# Order API Contract

## 1. Scope

Covers customer checkout from cart and order lookup, plus seller order-item
processing. This file defines only order-scoped endpoints.

Common conventions — base path `/api/v1`, `Authorization: Bearer <access-token>`,
UUID identifiers, ISO-8601 UTC timestamps, money as VND decimal strings,
zero-based pagination, `ApiResponse<T>` envelope, and shared error codes/HTTP
mapping — are defined in `..\..\API_CONTRACT.md` (§2) and are not repeated here.
Checkout/order shapes reuse `..\..\API_CONTRACT.md` (§6).

Status model: `OrderStatus = PENDING | PAID | DELIVERED | COMPLETED | CANCELLED`,
`OrderItemStatus = PENDING | CONFIRMED | CANCELLED`. `OrderItemStatus` tracks
seller processing only; delivery progress belongs to `ShipmentStatus`.

Endpoints marked `Predicted` are planned beyond the first MVP checkout slice
(see `..\..\API_CONTRACT.md` §10 Deferred contract).

## 2. Endpoints

### POST /api/v1/orders

Status: Confirmed — implemented in Phase 4 (customer checkout).

Create an order transactionally from the cart lines the authenticated customer selected.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

Request body:

```json
{
  "cartItemIds": ["cart-item-uuid-1", "cart-item-uuid-2"],
  "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
  "paymentMethod": "COD",
  "voucherCode": "SALE10"
}
```

| Field | Rule |
|---|---|
| `cartItemIds` | Required, non-empty, no duplicates. Ids of cart lines in the caller's own cart; only these lines are bought. |
| `shippingAddress` | Required, non-blank string, at most 1000 characters (address snapshot). The app may prefill it from the profile `address`. |
| `paymentMethod` | Required. `COD` or `VNPAY`. With `VNPAY`, checkout also creates the first `PENDING` VNPAY attempt and returns its `paymentUrl` (see `..\payment\API_CONTRACT.md`). |
| `voucherCode` | Optional, at most 64 characters, case-insensitive. Applies one platform voucher (see `..\voucher\API_CONTRACT.md`): consumes one use atomically and sets `discountAmount` to `discountPercent` % of the subtotal. |

Without `voucherCode`, `discountAmount` is `"0.00"` and `voucherCode` is `null` in the response. With it,
the discount is recalculated on the remaining lines whenever a line is cancelled later; the voucher use
is not given back (implemented in Phase 8).

Behavior (transactional, FR-ORD-01 – FR-ORD-08):

- Only the selected cart lines are bought; unselected lines stay in the cart.
- Validate every selected product is active and sufficiently stocked.
- Recalculate prices server-side; snapshot `productName`/`unitPrice` per item.
- Decrease stock with a conditional update so concurrent checkouts cannot oversell; save
  `discountAmount`/`shippingFee`/`totalAmount` snapshots; create one `PENDING` payment (COD, or
  VNPAY with a signed payment link) for `totalAmount`; remove the purchased lines from the cart.
- Any failure rolls back the whole checkout: no order, stock and cart unchanged.

Success: `201 Created`, `ApiResponse<OrderResponse>`. `data` payload:

```json
{
  "id": "order-uuid",
  "status": "PENDING",
  "paymentStatus": "PENDING",
  "paymentMethod": "COD",
  "paymentUrl": null,
  "createdAt": "2026-10-12T10:30:00Z",
  "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
  "shippingFee": "0.00",
  "discountAmount": "0.00",
  "totalAmount": "118.00",
  "voucherCode": null,
  "shipments": [],
  "items": [
    {
      "id": "order-item-uuid",
      "orderId": "order-uuid",
      "productId": "product-uuid",
      "productName": "Headphone",
      "unitPrice": "59.00",
      "quantity": 2,
      "status": "PENDING",
      "sellerId": "seller-uuid"
    }
  ]
}
```

Money is a VND decimal string; `shippingFee` is `"0.00"` in the first slice;
`totalAmount = item subtotal - discountAmount + shippingFee`, computed by the
backend. One order may contain items from multiple sellers (seller attribution
at `OrderItem` level).

`paymentUrl` is the signed VNPAY payment link when `paymentMethod` is `VNPAY`
and `null` for COD. `GET /api/v1/orders/{id}` always returns `null`; a new link
comes from `POST /api/v1/payments/vnpay/create`. A VNPAY order not paid within
24 hours of `createdAt` is cancelled automatically (FR-ORD-10). `shipments`
lists the order's shipments with the same shape as `ShipmentResponse` in
`..\shipment\API_CONTRACT.md`; it is empty at checkout.

Errors (common envelope):

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid `CUSTOMER` token. |
| `FORBIDDEN` | `403` | Authenticated actor is not `CUSTOMER`. |
| `VALIDATION_ERROR` | `400` | Empty or duplicate `cartItemIds`, blank or too long address, unknown `paymentMethod`. |
| `CART_ITEM_NOT_FOUND` | `404` | A selected id is not in the caller's cart (another customer's line is reported the same way). |
| `PRODUCT_INACTIVE` | `400` | A selected product is inactive/soft-deleted (same status as the cart endpoints). |
| `INSUFFICIENT_STOCK` | `409` | A selected line exceeds current stock. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown `voucherCode`. |
| `VOUCHER_EXPIRED` | `409` | The voucher is outside its validity window. |
| `VOUCHER_EXHAUSTED` | `409` | The voucher has no use left. |

Maps to FR-ORD-01, FR-ORD-02, FR-ORD-03, FR-ORD-04, FR-ORD-06, FR-ORD-07,
FR-ORD-08 (FR-ORD-05 voucher: Phase 8).

### GET /api/v1/orders

Status: Confirmed — implemented in Phase 4.

List orders owned by the authenticated customer, newest first.
Actor: `CUSTOMER`.

Query: `page` (default `0`), `size` (default `20`, backend-capped).

Success: `200 OK`, `ApiResponse<PageResponse<OrderSummaryResponse>>`. `data` payload:

```json
{
  "content": [
    {
      "id": "order-uuid",
      "status": "PENDING",
      "paymentStatus": "PENDING",
      "paymentMethod": "COD",
      "createdAt": "2026-10-12T10:30:00Z",
      "totalAmount": "118.00"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

Errors: `AUTHENTICATION_REQUIRED` (`401`).

Maps to FR-ORD-09.

### GET /api/v1/orders/{id}

Status: Confirmed — implemented in Phase 4.

Get one order owned by the authenticated customer.
Actor: `CUSTOMER`.

- `{id}`: order UUID.
- Returns `ApiResponse<OrderResponse>` with the same shape as `POST /api/v1/orders`.
- Ownership: an order that does not exist **or** does not belong to the caller
  returns `RESOURCE_NOT_FOUND` (`404`); the backend must not distinguish
  "another customer's order" from "missing order".

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown id or order owned by another customer. |

Maps to FR-ORD-09.

### PATCH /api/v1/orders/{id}/received

Status: Confirmed — implemented in Phase 7.

The customer confirms that the order was received, which closes the order
(`DELIVERED -> COMPLETED`). Actor: `CUSTOMER` (owner of the order). No request
body.

Rules (FR-ORD-13):

- Allowed only while the order is `DELIVERED` (every required shipment is
  delivered). Any other status is rejected.
- If the customer does not confirm, the system completes the order
  automatically 3 days after its last shipment was delivered (a background job
  checks periodically).
- Confirming receipt changes no payment: COD was already recorded as collected
  when the order became `DELIVERED`.

Success: `200 OK`, `ApiResponse<OrderResponse>` with `status: "COMPLETED"`.

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Authenticated actor is not `CUSTOMER`. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown id or order owned by another customer. |
| `VALIDATION_ERROR` | `400` | The order is not `DELIVERED`. |

Maps to FR-ORD-13.

### GET /api/v1/seller/order-items

Status: Confirmed — implemented in Phase 5.

Query order items owned by the authenticated seller, newest order first. The seller must be
`APPROVED` (checked in the database, like seller product management).
Actor: `SELLER` (`Authorization: Bearer <access-token>` required).

Query parameters (all optional): `status` (`PENDING | CONFIRMED | CANCELLED`),
`orderId` (UUID filter), `page` (default `0`), `size` (default `20`).

Enforces `orderItem.seller.id == authenticatedSeller.id`; never returns another
seller's lines.

Success: `200 OK`, `ApiResponse<PageResponse<OrderItemResponse>>`. `data` payload:

```json
{
  "content": [
    {
      "id": "order-item-uuid",
      "orderId": "order-uuid",
      "productId": "product-uuid",
      "productName": "Headphone",
      "unitPrice": "59.00",
      "quantity": 2,
      "status": "PENDING",
      "sellerId": "seller-uuid"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

Errors: `AUTHENTICATION_REQUIRED` (`401`); `FORBIDDEN` (`403`) when caller is
not `SELLER` or not `APPROVED`; `VALIDATION_ERROR` (`400`) for an unknown `status` value
or invalid `orderId`/`page`/`size`.

Maps to FR-FUL-01, FR-FUL-03.

### PATCH /api/v1/seller/order-items/{id}/confirm

Status: Confirmed — implemented in Phase 5.

Seller confirms one owned order item (`PENDING -> CONFIRMED`).
Actor: `SELLER`.

No request body. Success: `200 OK`, `ApiResponse<OrderItemResponse>` (same item
shape as above, `status: "CONFIRMED"`).

Rule (FR-FUL-05): a line of a VNPAY order can be confirmed only when the order is `PAID`. Before
that the line stays visible in the seller list but confirming it is refused. COD orders can be
confirmed right away.

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Caller is not an `APPROVED` `SELLER`, or `orderItem.seller.id != authenticatedSeller.id`. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown item id. |
| `VALIDATION_ERROR` | `400` | Item is not in `PENDING` (e.g. already `CONFIRMED`/`CANCELLED`), or the order is paid by VNPAY and not yet `PAID`. |

Maps to FR-FUL-02, FR-FUL-03, FR-FUL-04, FR-FUL-05.

### PATCH /api/v1/seller/order-items/{id}/cancel

Status: Confirmed — implemented in Phase 5; the prepayment check is implemented in Phase 6 and the
shipment check is implemented in Phase 7.

Seller cancels one owned order item before shipment assignment.
Actor: `SELLER`.

Request body:

```json
{
  "cancelReason": "Out of stock"
}
```

`cancelReason` is optional (`null` allowed, at most 1000 characters). The body itself may be omitted.

Rules:

- A line in `PENDING` or `CONFIRMED` can be cancelled; a `CANCELLED` line cannot (400).
- Cancelling restores the line quantity to product stock exactly once.
- Direct cancellation is allowed only before shipment assignment
  (FR-ORD-11). An item already mapped to a shipment must go through the atomic
  shipment-cancellation path, not this endpoint.
- Cancellation after `SHIPPED` or any later shipment state is rejected with
  `CANNOT_CANCEL_AFTER_SHIPMENT` (`409`).
- A line of an order that is already `PAID` cannot be cancelled
  (`CANNOT_CANCEL_AFTER_PAYMENT`, `409`) until a refund policy exists.
- Before payment completion the transaction recalculates `Order.totalAmount`
  and every still-`PENDING` COD payment `amount` from remaining non-cancelled
  snapshots; cancelled snapshots stay unchanged for history. A `PENDING` VNPAY
  attempt becomes `FAILED` instead, because its payment link carries the old
  amount; the customer pays the new total through the retry endpoint.
- When every line of the order is cancelled, the order becomes `CANCELLED`, its total becomes
  `0.00`, and every `PENDING` payment becomes `FAILED` (not collectible).
- The order row is locked during cancellation so two sellers cancelling lines of the same order
  cannot both recalculate from a stale total.

Success: `200 OK`, `ApiResponse<OrderItemResponse>` (`status: "CANCELLED"`).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Caller is not an `APPROVED` `SELLER`, or the line is not seller-owned. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown item id. |
| `VALIDATION_ERROR` | `400` | Item already `CANCELLED`, or `cancelReason` too long. |
| `CANNOT_CANCEL_AFTER_SHIPMENT` | `409` | Item already assigned to a shipment; cancel the shipment instead. |
| `CANNOT_CANCEL_AFTER_PAYMENT` | `409` | The order is already `PAID`; no refund policy exists yet. |

Maps to FR-ORD-11, FR-ORD-12, FR-FUL-02, FR-FUL-03, FR-FUL-04.
