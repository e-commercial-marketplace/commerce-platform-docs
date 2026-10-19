# Shipment API Contract

## 1. Scope

Covers seller shipment creation/management and customer shipment lookup. This
file defines only shipment-scoped endpoints.

Common conventions — base path `/api/v1`, `ApiResponse<T>` envelope, UUID
identifiers, ISO-8601 UTC timestamps, VND decimal strings, zero-based
pagination, and shared error codes/HTTP mapping — are defined in
`..\..\API_CONTRACT.md` (§2) and are not repeated here. The nested shipment
shape also appears in `..\..\API_CONTRACT.md` (§6 order detail).

Model: `DeliveryMethod = PLATFORM_DELIVERY | SELF_DELIVERY`;
`ShipmentStatus = PENDING | PREPARING | SHIPPED | DELIVERED | FAILED |
RETURNED | CANCELLED`. One shipment belongs to exactly one order and one
seller; all mapped items share that `orderId` and `sellerId`. MVP maps whole
order-item lines only (no partial/split/replacement shipments); one order item
belongs to at most one shipment.

All endpoints below are `Predicted` (see `..\..\API_CONTRACT.md` §10 Deferred
contract: seller shipment management and tracking UI are deferred from the
first MVP).

## 2. Endpoints

### POST /api/v1/seller/shipments — Predicted

Create a shipment from the seller's own confirmed order items.
Actor: `SELLER` (`Authorization: Bearer <access-token>` required).

Request body:

```json
{
  "orderId": "order-uuid",
  "deliveryMethod": "SELF_DELIVERY",
  "orderItemIds": ["order-item-uuid-1", "order-item-uuid-2"]
}
```

| Field | Rule |
|---|---|
| `orderId` | Required UUID of the parent order. |
| `deliveryMethod` | Required: `PLATFORM_DELIVERY` or `SELF_DELIVERY`. |
| `orderItemIds` | Required non-empty array of order-item UUIDs (whole lines only). |

Behavior (FR-SHIP-01 – FR-SHIP-04, FR-SHIP-13):

- Every `orderItemId` must be `CONFIRMED`, belong to `orderId`, and satisfy
  `orderItem.seller.id == authenticatedSeller.id`.
- All items share the same seller and order (`shipment.orderId ==
  orderItem.orderId`); mixed-seller or cross-order groups are rejected.
- An item already mapped to another shipment is rejected.
- New shipment starts as `PENDING` with `trackingCode/shippedAt/deliveredAt =
  null`.

Success: `201 Created`, `ApiResponse<ShipmentDto>`. `data` payload:

```json
{
  "id": "shipment-uuid",
  "orderId": "order-uuid",
  "sellerId": "seller-uuid",
  "deliveryMethod": "SELF_DELIVERY",
  "status": "PENDING",
  "trackingCode": null,
  "shippingFee": "0.00",
  "shippedAt": null,
  "deliveredAt": null,
  "items": [
    { "orderItemId": "order-item-uuid-1", "quantity": 2 }
  ]
}
```

`quantity` echoes the mapped `OrderItem.quantity` (whole-line mapping);
`shippingFee` is a VND decimal string (`"0.00"` in the first slice).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Caller not `SELLER`, or any item owned by another seller. |
| `VALIDATION_ERROR` | `400` | Empty `orderItemIds`, mixed order/seller, item not `CONFIRMED`, or item already shipped. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown order or order-item id. |

Maps to FR-SHIP-01, FR-SHIP-02, FR-SHIP-03, FR-SHIP-04, FR-SHIP-13.

### GET /api/v1/seller/shipments — Predicted

List shipments owned by the authenticated seller.
Actor: `SELLER`.

Query (all optional): `orderId` (UUID), `status` (`ShipmentStatus` value),
`page` (default `0`), `size` (default `20`).

Success: `200 OK`, `ApiResponse<PageResponse<ShipmentDto>>` (item shape as in
`POST /api/v1/seller/shipments`).

Errors: `AUTHENTICATION_REQUIRED` (`401`); `FORBIDDEN` (`403`) when caller is
not `SELLER`; `VALIDATION_ERROR` (`400`) for an unknown `status` value.

Maps to FR-SHIP-02, FR-SHIP-09.

### PATCH /api/v1/seller/shipments/{id}/status — Predicted

Advance one seller-owned shipment along the allowed state machine.
Actor: `SELLER`.

Request body:

```json
{
  "status": "SHIPPED",
  "trackingCode": "TRACK-001"
}
```

| Field | Rule |
|---|---|
| `status` | Required target `ShipmentStatus`. |
| `trackingCode` | Required when transitioning to `SHIPPED`; ignored/optional otherwise. |

Allowed transitions:

```text
PENDING -> PREPARING -> SHIPPED -> DELIVERED
SHIPPED -> FAILED -> RETURNED
PENDING/PREPARING -> CANCELLED (before payment completion only)
```

Field requirements (FR-SHIP-05, FR-SHIP-06):

- `SHIPPED` requires `trackingCode` and sets `shippedAt = now`.
- `DELIVERED` requires and sets `deliveredAt = now`.
- `FAILED` restores no stock. `RETURNED` is allowed only after the package
  reaches the seller warehouse and restores that shipment's shipped quantity
  exactly once.
- `CANCELLED` from `PENDING`/`PREPARING` is an atomic operation: marks mapped
  whole lines `CANCELLED` and releases unshipped stock exactly once; rejected
  after prepayment completion until a refund policy exists.
- Final `DELIVERED` transitions coordinate COD completion and the
  order-`DELIVERED` aggregate (all required non-cancelled shipments delivered;
  mixed outcomes stay per-shipment, FR-SHIP-07 – FR-SHIP-12).

Success: `200 OK`, `ApiResponse<ShipmentDto>` (same shape as creation,
reflecting the new status/timestamps).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Caller not `SELLER`, or shipment owned by another seller. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown shipment id. |
| `VALIDATION_ERROR` | `400` | Illegal transition, missing `trackingCode` for `SHIPPED`, or `CANCELLED` after payment completion. |

Maps to FR-SHIP-05, FR-SHIP-06, FR-SHIP-07, FR-SHIP-09, FR-SHIP-10, FR-SHIP-11,
FR-SHIP-12.

### GET /api/v1/orders/{orderId}/shipments — Predicted

List shipments for one order owned by the authenticated customer.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

- `{orderId}`: parent order UUID.
- Returns only shipments of that order; a customer must not query another
  customer's order (unknown or another customer's `orderId` yields `RESOURCE_NOT_FOUND`
  `404`).

Success: `200 OK`, `ApiResponse<List<ShipmentDto>>`:

```json
{
  "success": true,
  "data": [
    {
      "id": "shipment-uuid",
      "orderId": "order-uuid",
      "sellerId": "seller-uuid",
      "deliveryMethod": "SELF_DELIVERY",
      "status": "SHIPPED",
      "trackingCode": "TRACK-001",
      "shippingFee": "0.00",
      "shippedAt": "2026-10-13T09:00:00Z",
      "deliveredAt": null,
      "items": [
        { "orderItemId": "order-item-uuid", "quantity": 2 }
      ]
    }
  ],
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

 mixed seller outcomes are reported per shipment; the client must not collapse
them into a single order-level delivery status.

Errors: `AUTHENTICATION_REQUIRED` (`401`); `RESOURCE_NOT_FOUND` (`404`) when
the order is unknown or not owned by the caller.

Maps to FR-SHIP-08, FR-SHIP-09, FR-SHIP-10.
