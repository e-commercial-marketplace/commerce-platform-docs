# Voucher API Contract

## 1. Scope

Covers customer voucher validation/application and platform voucher
administration. This file defines only voucher-scoped endpoints.

Common conventions — base path `/api`, `ApiResponse<T>` envelope, UUID
identifiers, ISO-8601 UTC timestamps, money as VND decimal strings, zero-based
pagination, and shared error codes/HTTP mapping — are defined in
`..\..\API_CONTRACT.md` (§2) and are not repeated here.

Model: `Voucher(code, description, discountPercent, validFrom, validTo,
usageLimit, timesUsed)`. `orders.voucherCode` is optional; at most one voucher
per order. Consumption must be atomic (conditional update / optimistic locking)
so concurrent final uses cannot exceed `usageLimit`.

All endpoints below are `Predicted` (see `..\..\API_CONTRACT.md` §10 Deferred
contract: voucher validation/application and Admin operations are deferred from
the first MVP).

## 2. Customer endpoints

### GET /api/vouchers/{code}/validate — Predicted

Validate a voucher code without applying it.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

- `{code}`: voucher business code (e.g. `SALE10`).
- Read-only: does not increment `timesUsed` or attach the voucher to any order.

Success: `200 OK`, `ApiResponse<VoucherValidityDto>`. `data` payload:

```json
{
  "code": "SALE10",
  "description": "10% off platform campaign",
  "discountPercent": 10,
  "validFrom": "2026-10-01T00:00:00Z",
  "validTo": "2026-11-01T00:00:00Z",
  "usageLimit": 100,
  "timesUsed": 42,
  "valid": true,
  "reason": null
}
```

`validFrom`/`validTo` are ISO-8601 UTC; the window is
`validFrom <= now < validTo`.

Errors (common envelope):

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid `CUSTOMER` token. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown `code`. |
| `VOUCHER_EXPIRED` | `409` | Outside `[validFrom, validTo)`. |
| `VOUCHER_EXHAUSTED` | `409` | `timesUsed >= usageLimit`. |

Maps to FR-VCH-02, FR-VCH-03.

### POST /api/orders/{id}/voucher — Predicted

Apply one voucher to the customer's own order.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

Request body:

```json
{
  "code": "SALE10"
}
```

Behavior (FR-VCH-01 – FR-VCH-04):

- Order must exist, belong to the caller, and be in a mutable pre-payment state.
- At most one voucher per order: if the order already has a `voucherCode`,
  reject with `VOUCHER_ALREADY_APPLIED`.
- Validate expiry (`validFrom <= now < validTo`) and exhaustion
  (`timesUsed < usageLimit`).
- Consume atomically inside the order transaction, e.g.
  `UPDATE vouchers SET times_used = times_used + 1 WHERE code = ? AND
  valid_from <= CURRENT_TIMESTAMP AND valid_to > CURRENT_TIMESTAMP AND
  times_used < usage_limit`; success requires exactly one row updated,
  otherwise reject. Recompute `discountAmount`/`totalAmount` snapshots from the
  order-item snapshots (never reread current product prices).

Success: `200 OK`, `ApiResponse<OrderDto>` (order shape per the order contract,
with `voucherCode` and updated snapshots):

```json
{
  "id": "order-uuid",
  "status": "PENDING",
  "paymentStatus": "PENDING",
  "paymentMethod": "COD",
  "createdAt": "2026-10-12T10:30:00Z",
  "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
  "shippingFee": "0.00",
  "discountAmount": "11.80",
  "totalAmount": "106.20",
  "voucherCode": "SALE10",
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

Money is a VND decimal string.

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown order id, another customer's order, or unknown voucher `code`. |
| `VALIDATION_ERROR` | `400` | Missing `code`, or order no longer voucher-mutable. |
| `VOUCHER_ALREADY_APPLIED` | `409` | Order already has a voucher. |
| `VOUCHER_EXPIRED` | `409` | Voucher outside validity window / atomic update affected 0 rows due to expiry. |
| `VOUCHER_EXHAUSTED` | `409` | `usageLimit` reached / atomic update affected 0 rows due to exhaustion. |

Maps to FR-VCH-01, FR-VCH-02, FR-VCH-03, FR-VCH-04.

## 3. Admin endpoints — Predicted

Prefix: `/api/admin/vouchers`. Actor: `ADMIN`
(`Authorization: Bearer <access-token>` required) for all four endpoints.
`FORBIDDEN` (`403`) when the caller is not `ADMIN`.

### GET /api/admin/vouchers — Predicted

List platform vouchers (paginated). Query: `page` (default `0`), `size`
(default `20`).

Success: `200 OK`, `ApiResponse<PageResponse<VoucherDto>>`. Item `data` shape:

```json
{
  "id": "voucher-uuid",
  "code": "SALE10",
  "description": "10% off platform campaign",
  "discountPercent": 10,
  "validFrom": "2026-10-01T00:00:00Z",
  "validTo": "2026-11-01T00:00:00Z",
  "usageLimit": 100,
  "timesUsed": 42
}
```

Errors: `AUTHENTICATION_REQUIRED` (`401`); `FORBIDDEN` (`403`).

### POST /api/admin/vouchers — Predicted

Create a platform voucher.

Request body:

```json
{
  "code": "SALE10",
  "description": "10% off platform campaign",
  "discountPercent": 10,
  "validFrom": "2026-10-01T00:00:00Z",
  "validTo": "2026-11-01T00:00:00Z",
  "usageLimit": 100
}
```

Rules: `code` unique/non-blank; `discountPercent` in `1..100`;
`validFrom < validTo`; `usageLimit >= 1`. `timesUsed` starts at `0`.

Success: `201 Created`, `ApiResponse<VoucherDto>` (shape above).

Errors: `AUTHENTICATION_REQUIRED` (`401`); `FORBIDDEN` (`403`);
`VALIDATION_ERROR` (`400`) for constraint violations or duplicate `code`.

### PATCH /api/admin/vouchers/{id} — Predicted

Update a platform voucher (`{id}`: voucher UUID).

Request body (all fields optional; only present fields change):

```json
{
  "description": "10% off, extended",
  "discountPercent": 15,
  "validFrom": "2026-10-01T00:00:00Z",
  "validTo": "2026-11-15T00:00:00Z",
  "usageLimit": 200
}
```

`code` is immutable after creation. Reducing `usageLimit` below current
`timesUsed` is rejected. Shortening `validTo` to the past effectively expires
the voucher.

Success: `200 OK`, `ApiResponse<VoucherDto>`.

Errors: `AUTHENTICATION_REQUIRED` (`401`); `FORBIDDEN` (`403`);
`RESOURCE_NOT_FOUND` (`404`) for unknown `{id}`; `VALIDATION_ERROR` (`400`)
for constraint violations.

### DELETE /api/admin/vouchers/{id} — Predicted

Expire a platform voucher (`{id}`: voucher UUID). This is an expiry operation,
not a physical delete: it sets `validTo = now` so in-flight validation and the
atomic consumption guard reject further uses, while history (`timesUsed`,
applied orders) is preserved.

Success: `200 OK`, `ApiResponse<VoucherDto>` reflecting the expired
`validTo`.

Errors: `AUTHENTICATION_REQUIRED` (`401`); `FORBIDDEN` (`403`);
`RESOURCE_NOT_FOUND` (`404`) for unknown `{id}`.
