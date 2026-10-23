# Shipment API Contract

## 1. Scope

Covers seller shipment creation/management and the carrier list managed by
admin. Customers read shipments inside the order detail. This file defines only
shipment-scoped endpoints.

Common conventions — base path `/api/v1`, `ApiResponse<T>` envelope, UUID
identifiers, ISO-8601 UTC timestamps, VND decimal strings, zero-based
pagination, and shared error codes/HTTP mapping — are defined in
`..\..\API_CONTRACT.md` (§2) and are not repeated here. The nested shipment
shape also appears in `..\..\API_CONTRACT.md` (§6 order detail).

Model: `DeliveryMethod = PLATFORM_DELIVERY | SELF_DELIVERY`. A
`PLATFORM_DELIVERY` shipment names one carrier from the admin-managed list; a
`SELF_DELIVERY` shipment (the seller delivers it, for example a nearby order)
has no carrier. The seller decides; the system does not measure distance.
`ShipmentStatus = PENDING | PREPARING | SHIPPED | DELIVERED | FAILED |
RETURNED | CANCELLED`. One shipment belongs to exactly one order and one
seller; all mapped items share that `orderId` and `sellerId`. MVP maps whole
order-item lines only (no partial/split/replacement shipments); one order item
belongs to at most one shipment.

Status: Confirmed — implemented in Phase 7.

## 2. Endpoints

### POST /api/v1/seller/shipments — Confirmed

Create a shipment from the seller's own confirmed order items.
Actor: `SELLER` (`Authorization: Bearer <access-token>` required).

Request body:

```json
{
  "orderId": "order-uuid",
  "deliveryMethod": "PLATFORM_DELIVERY",
  "carrierId": "carrier-uuid",
  "orderItemIds": ["order-item-uuid-1", "order-item-uuid-2"]
}
```

| Field | Rule |
|---|---|
| `orderId` | Required UUID of the parent order. |
| `deliveryMethod` | Required: `PLATFORM_DELIVERY` or `SELF_DELIVERY`. |
| `carrierId` | Required with `PLATFORM_DELIVERY` and must be an active carrier; must be absent or `null` with `SELF_DELIVERY`. |
| `orderItemIds` | Required non-empty array of order-item UUIDs (whole lines only). |

Behavior (FR-SHIP-01 – FR-SHIP-04, FR-SHIP-13):

- Every `orderItemId` must be `CONFIRMED`, belong to `orderId`, and satisfy
  `orderItem.seller.id == authenticatedSeller.id`.
- All items share the same seller and order (`shipment.orderId ==
  orderItem.orderId`); mixed-seller or cross-order groups are rejected.
- An item already mapped to another shipment is rejected.
- A seller has at most one shipment per order: if one already exists for this
  order, the request is rejected.
- `carrierId` follows the rule in the table above (FR-SHIP-01, FR-SHIP-14).
- New shipment starts as `PENDING` with `trackingCode/shippedAt/deliveredAt =
  null`.

Success: `201 Created`, `ApiResponse<ShipmentResponse>`. `data` payload:

```json
{
  "id": "shipment-uuid",
  "orderId": "order-uuid",
  "sellerId": "seller-uuid",
  "deliveryMethod": "PLATFORM_DELIVERY",
  "carrier": { "id": "carrier-uuid", "name": "GHN" },
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
`shippingFee` is a VND decimal string (`"0.00"` in the first slice). `carrier`
is `null` for `SELF_DELIVERY`; a carrier that admin later deactivates still
appears on the shipments that used it.

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Caller not `SELLER`, or any item owned by another seller. |
| `VALIDATION_ERROR` | `400` | Empty `orderItemIds`, mixed order/seller, item not `CONFIRMED`, item already shipped, the seller already has a shipment for this order, or a missing, unknown, inactive or unexpected `carrierId`. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown order or order-item id. |

Maps to FR-SHIP-01, FR-SHIP-02, FR-SHIP-03, FR-SHIP-04, FR-SHIP-13, FR-SHIP-14.

### GET /api/v1/seller/shipments — Confirmed

List shipments owned by the authenticated seller.
Actor: `SELLER`.

Query (all optional): `orderId` (UUID), `status` (`ShipmentStatus` value),
`page` (default `0`), `size` (default `20`).

Success: `200 OK`, `ApiResponse<PageResponse<ShipmentResponse>>` (item shape as in
`POST /api/v1/seller/shipments`).

Errors: `AUTHENTICATION_REQUIRED` (`401`); `FORBIDDEN` (`403`) when caller is
not `SELLER`; `VALIDATION_ERROR` (`400`) for an unknown `status` value.

Maps to FR-SHIP-02, FR-SHIP-09.

### PATCH /api/v1/seller/shipments/{id}/status — Confirmed

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

Success: `200 OK`, `ApiResponse<ShipmentResponse>` (same shape as creation,
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

### Customer view

Customers read the shipments of their own order in `GET /api/v1/orders/{id}`
(the `shipments` array of `OrderResponse`, see `..\order\API_CONTRACT.md`);
there is no separate shipment endpoint. Mixed seller outcomes are reported per
shipment; the client must not collapse them into a single order-level delivery
status.

Maps to FR-SHIP-08, FR-SHIP-09, FR-SHIP-10.

## 3. Carriers

Status: Confirmed — implemented in Phase 7. Admin manages the carriers that
`PLATFORM_DELIVERY` shipments can use (FR-SHIP-14). A carrier is never deleted:
deactivating it hides it from sellers but keeps it on the shipments that used
it. Carrier names are unique, ignoring case.

`CarrierResponse` `data` shape:

```json
{
  "id": "carrier-uuid",
  "name": "GHN",
  "active": true
}
```

### GET /api/v1/admin/carriers

Actor: `ADMIN`. Returns every carrier, active or not, sorted by name:
`200 OK`, `ApiResponse<List<CarrierResponse>>`.

### POST /api/v1/admin/carriers

Actor: `ADMIN`. Body `{ "name": "GHN" }` (required, non-blank, at most 100
characters). Creates an active carrier: `201 Created`,
`ApiResponse<CarrierResponse>`.

### PATCH /api/v1/admin/carriers/{id}

Actor: `ADMIN`. Body `{ "name": "Giao Hang Nhanh", "active": false }`; each
field is optional (`null` keeps the current value), `name` must be non-blank
when sent. `200 OK`, `ApiResponse<CarrierResponse>`.

### GET /api/v1/seller/carriers

Actor: approved `SELLER`. Returns only active carriers, sorted by name:
`200 OK`, `ApiResponse<List<CarrierResponse>>`.

Errors for the carrier endpoints:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Wrong role, or a seller that is not `APPROVED`. |
| `VALIDATION_ERROR` | `400` | Blank or too long `name`, or a name already used by another carrier. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown carrier id (`PATCH`). |

Maps to FR-SHIP-14.
