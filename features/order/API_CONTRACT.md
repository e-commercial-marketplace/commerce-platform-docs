# Order API Contract

## 1. Scope

Covers customer checkout from cart and order lookup, plus seller order-item
processing. This file defines only order-scoped endpoints.

Common conventions — base path `/api/v1`, `Authorization: Bearer <access-token>`,
UUID identifiers, ISO-8601 UTC timestamps, money as VND decimal strings,
zero-based pagination, `ApiResponse<T>` envelope, and shared error codes/HTTP
mapping — are defined in `..\..\API_CONTRACT.md` (§2) and are not repeated here.
Checkout/order shapes reuse `..\..\API_CONTRACT.md` (§6).

Status model: `OrderStatus = PENDING | PAID | DELIVERED | CANCELLED`,
`OrderItemStatus = PENDING | CONFIRMED | CANCELLED`. `OrderItemStatus` tracks
seller processing only; delivery progress belongs to `ShipmentStatus`.

Endpoints marked `Predicted` are planned beyond the first MVP checkout slice
(see `..\..\API_CONTRACT.md` §10 Deferred contract).

## 2. Endpoints

### POST /api/v1/orders

Create an order transactionally from the authenticated customer's cart.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

Request body:

```json
{
  "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
  "paymentMethod": "COD",
  "voucherCode": null
}
```

| Field | Rule |
|---|---|
| `shippingAddress` | Required, non-blank string (address snapshot). |
| `paymentMethod` | Required. `COD` in MVP; `VNPAY` reserved (see payment contract). |
| `voucherCode` | Optional, `null` when unused. At most one voucher per order. |

Behavior (transactional, FR-ORD-01 – FR-ORD-08):

- Validate every cart product is active and sufficiently stocked.
- Validate voucher validity/usage when `voucherCode` is present.
- Recalculate prices server-side; snapshot `productName`/`unitPrice` per item.
- Reserve/decrease stock, save `discountAmount`/`shippingFee`/`totalAmount`
  snapshots, clear purchased cart items.

Success: `201 Created`, `ApiResponse<OrderDto>`. `data` payload:

```json
{
  "id": "order-uuid",
  "status": "PENDING",
  "paymentStatus": "PENDING",
  "paymentMethod": "COD",
  "createdAt": "2026-10-12T10:30:00Z",
  "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
  "shippingFee": "0.00",
  "discountAmount": "0.00",
  "totalAmount": "118.00",
  "voucherCode": null,
  "items": [
    {
      "id": "order-item-uuid",
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

Errors (common envelope):

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid `CUSTOMER` token. |
| `VALIDATION_ERROR` | `400` | Blank address, unknown `paymentMethod`, invalid voucher usage. |
| `EMPTY_CART` | `409` | Cart has no purchasable items. |
| `PRODUCT_INACTIVE` | `409` | Any cart product is inactive/soft-deleted. |
| `INSUFFICIENT_STOCK` | `409` | Any line exceeds current stock. |

Maps to FR-ORD-01, FR-ORD-02, FR-ORD-03, FR-ORD-04, FR-ORD-05, FR-ORD-06,
FR-ORD-07, FR-ORD-08.

### GET /api/v1/orders

List orders owned by the authenticated customer.
Actor: `CUSTOMER`.

Query: `page` (default `0`), `size` (default `20`, backend-capped).

Success: `200 OK`, `ApiResponse<PageResponse<OrderSummaryDto>>`. `data` payload:

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

Get one order owned by the authenticated customer.
Actor: `CUSTOMER`.

- `{id}`: order UUID.
- Returns `ApiResponse<OrderDto>` with the same shape as `POST /api/v1/orders`.
- Ownership: an order that does not exist **or** does not belong to the caller
  returns `RESOURCE_NOT_FOUND` (`404`); the backend must not distinguish
  "another customer's order" from "missing order".

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown id or order owned by another customer. |

Maps to FR-ORD-09.

### GET /api/v1/seller/order-items — Predicted

Query order items owned by the authenticated seller.
Actor: `SELLER` (`Authorization: Bearer <access-token>` required).

Query parameters (all optional): `status` (`PENDING | CONFIRMED | CANCELLED`),
`orderId` (UUID filter), `page` (default `0`), `size` (default `20`).

Enforces `orderItem.seller.id == authenticatedSeller.id`; never returns another
seller's lines.

Success: `200 OK`, `ApiResponse<PageResponse<OrderItemDto>>`. `data` payload:

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
not `SELLER`; `VALIDATION_ERROR` (`400`) for an unknown `status` value.

Maps to FR-FUL-01, FR-FUL-03.

### PATCH /api/v1/seller/order-items/{id}/confirm — Predicted

Seller confirms one owned order item (`PENDING -> CONFIRMED`).
Actor: `SELLER`.

No request body. Success: `200 OK`, `ApiResponse<OrderItemDto>` (same item
shape as above, `status: "CONFIRMED"`).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Caller is not `SELLER`, or `orderItem.seller.id != authenticatedSeller.id`. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown item id (or not visible to caller). |
| `VALIDATION_ERROR` | `400` | Item is not in `PENDING` (e.g. already `CONFIRMED`/`CANCELLED`). |

Maps to FR-FUL-02, FR-FUL-03, FR-FUL-04.

### PATCH /api/v1/seller/order-items/{id}/cancel — Predicted

Seller cancels one owned order item before shipment assignment.
Actor: `SELLER`.

Request body:

```json
{
  "cancelReason": "Out of stock"
}
```

`cancelReason` is optional (`null` allowed).

Rules:

- Direct cancellation is allowed only before shipment assignment
  (FR-ORD-11). An item already mapped to a shipment must go through the atomic
  shipment-cancellation path, not this endpoint.
- Cancellation after `SHIPPED` or any later shipment state is rejected with
  `CANNOT_CANCEL_AFTER_SHIPMENT` (`409`).
- Before payment completion the transaction recalculates `Order.totalAmount`
  and every still-`PENDING` payment `amount` from remaining non-cancelled
  snapshots; cancelled snapshots stay unchanged for history.

Success: `200 OK`, `ApiResponse<OrderItemDto>` (`status: "CANCELLED"`).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Not seller-owned. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown item id. |
| `VALIDATION_ERROR` | `400` | Item not cancellable in its current `OrderItemStatus`. |
| `CANNOT_CANCEL_AFTER_SHIPMENT` | `409` | Item already assigned/shipped via a shipment. |
| `CANNOT_CANCEL_AFTER_PAYMENT` | `409` | Prepayment already completed and no refund policy applies. |

Maps to FR-ORD-11, FR-ORD-12, FR-FUL-02, FR-FUL-03, FR-FUL-04.
