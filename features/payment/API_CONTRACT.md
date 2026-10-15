# Payment API Contract

## 1. Scope

Covers VNPAY payment creation and callback handling for marketplace orders. COD
creation/completion rules are defined by the order/shipment flows; this file
defines only the VNPAY endpoints.

Common conventions — `ApiResponse<T>` envelope, UUID identifiers, money as VND
decimal strings, ISO-8601 UTC timestamps, and shared error codes/HTTP mapping —
are defined in `..\..\API_CONTRACT.md` (§2) and are not repeated here.

Model: one `Order` may have many `Payment` attempts. `PaymentMethod = COD |
VNPAY`; `PaymentStatus = PENDING | COMPLETED | FAILED | REFUNDED`.
`transactionId` is required and unique once the gateway returns one (`NULL`
before that). Amounts use `BigDecimal` server-side, VND decimal strings on the
wire.

Both endpoints below are `Predicted` (see `..\..\API_CONTRACT.md` §10 Deferred
contract: VNPAY request/callback payloads are not frozen for the first MVP).

## 2. Endpoints

### POST /api/payments/vnpay/create — Predicted

Create a `PENDING` VNPAY payment attempt for the customer's own payable order
and return redirect parameters for the provider.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

Request body:

```json
{
  "orderId": "order-uuid",
  "returnUrl": "myapp://payments/vnpay/return"
}
```

| Field | Rule |
|---|---|
| `orderId` | Required UUID of a `PENDING`-payable order owned by the caller. |
| `returnUrl` | Required HTTPS/app-link URL the provider redirects to after payment. |

Behavior (FR-PAY-01, FR-PAY-03, FR-PAY-07):

- Verify order ownership and that the order is still payable (not
  `DELIVERED`/`CANCELLED`, no `COMPLETED` payment already recorded).
- Create a new `Payment(method=VNPAY, status=PENDING, amount=<current order
  payable>)` row; never reuse/overwrite a prior attempt.
- Allocate a unique `transactionId`/reference for this attempt and sign the
  provider request per VNPAY spec (keys stay on the backend).

Success: `200 OK`, `ApiResponse<VnpayCreateResponse>`. `data` payload:

```json
{
  "paymentId": "payment-uuid",
  "orderId": "order-uuid",
  "amount": "118.00",
  "status": "PENDING",
  "transactionId": "VNPAY-TXN-000001",
  "paymentUrl": "https://pay.vnpay.vn/payment?txnRef=VNPAY-TXN-000001&...",
  "redirectParams": {
    "vnp_TxnRef": "VNPAY-TXN-000001",
    "vnp_Amount": "1180000",
    "vnp_ReturnUrl": "myapp://payments/vnpay/return"
  },
  "createdAt": "2026-10-12T10:30:00Z"
}
```

`amount` is a VND decimal string. `paymentUrl`/`redirectParams` are opaque
provider inputs for the client redirect; the client must not construct or sign
them.

Errors (common envelope):

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid `CUSTOMER` token. |
| `VALIDATION_ERROR` | `400` | Missing/invalid `orderId` or `returnUrl`. |
| `RESOURCE_NOT_FOUND` | `404` | Order unknown or not owned by caller. |
| `ORDER_NOT_PAYABLE` | `409` | Order already paid/delivered/cancelled or already has a `COMPLETED` payment. |

Maps to FR-PAY-01, FR-PAY-03, FR-PAY-06, FR-PAY-07.

### POST /api/payments/vnpay/callback — Predicted

Handle the VNPAY server/redirect callback. Actor: `PUBLIC` (no bearer token;
called by the payment provider / return redirect).

Mandatory gateway authentication: the backend MUST validate the
checksum/signature on every callback before any state change, using the
provider secret stored server-side. An invalid signature rejects the callback
without mutating order/payment state.

Request body (gateway fields forwarded as JSON):

```json
{
  "transactionId": "VNPAY-TXN-000001",
  "orderId": "order-uuid",
  "amount": "118.00",
  "responseCode": "00",
  "transactionStatus": "00",
  "secureHash": "provider-signature-hex"
}
```

| Field | Rule |
|---|---|
| `transactionId` | Required, unique gateway reference; idempotency key. |
| `orderId` | Required order UUID the attempt belongs to. |
| `amount` | Required VND decimal string; must match the attempt `amount`. |
| `responseCode` / `transactionStatus` | Provider result codes (`00` = success). |
| `secureHash` | Required provider checksum/signature over the callback fields. |

Behavior (FR-PAY-03 – FR-PAY-07):

- Validate checksum/signature first; reject invalid callbacks.
- Idempotent by `transactionId`: a repeated callback for an already-processed
  `transactionId` returns the current payment state with `200 OK` and performs
  no duplicate transition.
- Success codes mark the attempt `COMPLETED` (`paidAt = now`) and coordinate
  the order aggregate per payment rules; failure codes mark the attempt
  `FAILED` only.
- A `FAILED` attempt MUST NOT destroy/modify the order; the order stays payable
  and a retry creates another `Payment` record via
  `POST /api/payments/vnpay/create`.

Success: `200 OK`, `ApiResponse<PaymentDto>`. `data` payload:

```json
{
  "id": "payment-uuid",
  "orderId": "order-uuid",
  "amount": "118.00",
  "method": "VNPAY",
  "status": "COMPLETED",
  "transactionId": "VNPAY-TXN-000001",
  "paidAt": "2026-10-12T11:00:00Z"
}
```

`paidAt` is `null` while `status` is `PENDING`/`FAILED`.

Errors (common envelope; failed payment attempts are recorded as `FAILED`
state, not as HTTP errors, unless the callback itself is invalid):

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | `400` | Malformed body, amount mismatch, or invalid checksum/signature. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown `orderId` or `transactionId` with no matching attempt. |
| `PAYMENT_FAILED` | `409` | Provider returned a failure code (also persisted as attempt `FAILED`). |
| `ORDER_NOT_PAYABLE` | `409` | Order already completed/cancelled; callback recorded but no transition. |

Maps to FR-PAY-03, FR-PAY-04, FR-PAY-05, FR-PAY-06, FR-PAY-07.
