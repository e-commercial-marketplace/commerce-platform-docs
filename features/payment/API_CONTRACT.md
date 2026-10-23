# Payment API Contract

## 1. Scope

Covers VNPAY payment for marketplace orders: the retry endpoint used by the
customer, and the two provider-facing endpoints (IPN and Return URL). The first
VNPAY attempt is created by checkout (`POST /api/v1/orders` with
`"paymentMethod": "VNPAY"`, see `..\order\API_CONTRACT.md`). COD
creation/completion rules are defined by the order/shipment flows.

Common conventions — `ApiResponse<T>` envelope, UUID identifiers, money as VND
decimal strings, ISO-8601 UTC timestamps, and shared error codes/HTTP mapping —
are defined in `..\..\API_CONTRACT.md` (§2) and are not repeated here. The IPN
and Return endpoints are the only exceptions to the envelope because their
response format is set by VNPAY.

Model: one `Order` may have many `Payment` attempts. `PaymentMethod = COD |
VNPAY`; `PaymentStatus = PENDING | COMPLETED | FAILED | REFUNDED`. For VNPAY,
`transactionId` holds the merchant reference sent as `vnp_TxnRef` (32 hex
characters, unique, assigned when the attempt is created). COD attempts keep
`transactionId = NULL`. Amounts use `BigDecimal` server-side, VND decimal strings
on the wire.

Status: Confirmed — implemented in Phase 6 against the VNPAY sandbox (payment
API version `2.1.0`, `HMAC-SHA512` signature).

## 2. Payment rules

- A VNPAY attempt starts `PENDING` and ends `COMPLETED` or `FAILED`; a failed
  attempt never destroys the order (FR-PAY-03, FR-PAY-06).
- A successful result marks the attempt `COMPLETED` (`paidAt = now`) and the
  order `PAID` in the same transaction.
- A retry creates a new attempt; every still-`PENDING` VNPAY attempt of the order
  is marked `FAILED` first, so at most one VNPAY link is payable at a time.
- When the order total changes before payment (an item is cancelled), every
  `PENDING` VNPAY attempt becomes `FAILED`, because its payment link carries the
  old amount. The customer pays again with the new total through the retry
  endpoint. `PENDING` COD attempts receive the new amount instead.
- A VNPAY order not paid within 24 hours of `Order.createdAt` is cancelled by a
  background job that runs every 5 minutes: all remaining lines are cancelled
  with reason `Payment deadline expired`, their stock is restored exactly once,
  and pending attempts become `FAILED` (FR-ORD-10).
- A VNPAY payment link expires 15 minutes after it is created (`vnp_ExpireDate`), and never later
  than the 24-hour deadline of the order, so a payment cannot arrive after the order was cancelled.
- When an order has several attempts, the order responses show the current one: the `COMPLETED`
  attempt if there is one, otherwise the `PENDING` one, otherwise the last.
- Known limitation: if VNPAY reports success for an attempt that is already
  `FAILED` (superseded by a retry, invalidated by a total change, or its order
  was cancelled), the backend changes nothing and logs a warning with the
  `vnp_TxnRef` so the payment can be refunded manually. Automatic refunds are
  out of scope.

## 3. Endpoints

### POST /api/v1/payments/vnpay/create

Create a new VNPAY attempt for the caller's own unpaid VNPAY order (retry after
a failed, abandoned, or invalidated attempt) and return the payment link.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

Request body:

```json
{
  "orderId": "order-uuid"
}
```

| Field | Rule |
|---|---|
| `orderId` | Required UUID of an order owned by the caller. |

The return URL is configured on the server (`app.vnpay.return-url`); the client
does not send one, so the endpoint cannot be used as an open redirect.

Behavior (FR-PAY-03, FR-PAY-06, FR-PAY-07):

- Lock the order and check it is owned by the caller, still `PENDING`, paid by
  VNPAY, and inside its 24-hour deadline.
- Mark every `PENDING` VNPAY attempt `FAILED`, then create
  `Payment(method=VNPAY, status=PENDING, amount=<current order total>)` with a
  new unique `transactionId`.
- Sign the provider request on the backend (the hash secret never leaves the
  server) and return the payment link.

Success: `200 OK`, `ApiResponse<VnpayPaymentResponse>`. `data` payload:

```json
{
  "paymentId": "payment-uuid",
  "orderId": "order-uuid",
  "amount": "34990000.00",
  "status": "PENDING",
  "paymentUrl": "https://sandbox.vnpayment.vn/paymentv2/vpcpay.html?vnp_Amount=3499000000&...&vnp_SecureHash=..."
}
```

`paymentUrl` is opaque: the client opens it as is and must not build or sign it.

Errors (common envelope):

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Authenticated actor is not `CUSTOMER`. |
| `VALIDATION_ERROR` | `400` | Missing or malformed `orderId`. |
| `RESOURCE_NOT_FOUND` | `404` | Order unknown or not owned by the caller. |
| `ORDER_NOT_PAYABLE` | `409` | Order is not `PENDING` (already paid, delivered, or cancelled), is a COD order, or its 24-hour deadline has passed. |

Maps to FR-PAY-01, FR-PAY-03, FR-PAY-06, FR-PAY-07.

### GET /api/v1/payments/vnpay/ipn

Instant Payment Notification: VNPAY calls this endpoint server-to-server with
the payment result. Actor: `PUBLIC` (no bearer token; authenticated by the
VNPAY signature). The IPN URL is registered in the VNPAY merchant portal and
must be publicly reachable (for example the Render deployment).

Query: every `vnp_*` field sent by VNPAY, including `vnp_TxnRef`,
`vnp_Amount` (amount × 100), `vnp_ResponseCode`, `vnp_TransactionStatus`,
`vnp_TransactionNo`, and `vnp_SecureHash`.

Processing (FR-PAY-03 – FR-PAY-07), in this order:

1. Recompute the `HMAC-SHA512` signature over the sorted `vnp_*` fields
   (excluding `vnp_SecureHash` and `vnp_SecureHashType`) and compare it with
   `vnp_SecureHash`. A mismatch changes nothing.
2. Find the attempt by `vnp_TxnRef`; lock its order.
3. Check `vnp_Amount` equals the attempt `amount` × 100.
4. Idempotency: an attempt that is no longer `PENDING` is not changed again.
5. `vnp_ResponseCode = 00` and `vnp_TransactionStatus = 00` mark the attempt
   `COMPLETED` and the order `PAID`; any other code marks the attempt `FAILED`
   and leaves the order payable.

Response: `200 OK` with the JSON body VNPAY expects (not wrapped in
`ApiResponse`):

```json
{
  "RspCode": "00",
  "Message": "Confirm Success"
}
```

| `RspCode` | Meaning |
|---|---|
| `00` | Result recorded (success or failure). |
| `01` | No attempt with this `vnp_TxnRef`. |
| `02` | Attempt already processed; nothing changed. |
| `04` | Amount does not match the attempt. |
| `97` | Invalid signature. |
| `99` | Unexpected error. |

Maps to FR-PAY-03, FR-PAY-04, FR-PAY-05, FR-PAY-07.

### GET /api/v1/payments/vnpay/return

Return URL: the customer's browser is redirected here by VNPAY after the
payment page. Actor: `PUBLIC` (authenticated by the VNPAY signature).

Query: the same `vnp_*` fields as the IPN.

Processing: exactly the same steps as the IPN, through the same idempotent
handler, so the payment is recorded even when the IPN cannot reach the backend
(local development). Whichever of IPN and Return arrives first records the
result; the other one sees an already-processed attempt (FR-PAY-08).

Response: `302 Found` to the configured app URL (`app.vnpay.app-redirect-url`):

```text
<app-redirect-url>?orderId=order-uuid&status=COMPLETED
```

| Query | Meaning |
|---|---|
| `orderId` | Order of the attempt; absent when the attempt is unknown or the signature is invalid. |
| `status` | `COMPLETED`, `FAILED`, or `INVALID` (signature, amount, or reference check failed). |

The app must read the final state from `GET /api/v1/orders/{id}`; the redirect
query is only a hint for which screen to show.

Maps to FR-PAY-04, FR-PAY-05, FR-PAY-08.
