# Voucher API Contract

## 1. Scope

Covers platform voucher administration and the customer validity check. A
voucher is applied at checkout through the optional `voucherCode` field of
`POST /api/v1/orders` (see `..\order\API_CONTRACT.md`); there is no separate
"apply voucher" endpoint.

Common conventions — base path `/api/v1`, `ApiResponse<T>` envelope, ISO-8601
UTC timestamps, money as VND decimal strings, zero-based pagination, and shared
error codes/HTTP mapping — are defined in `..\..\API_CONTRACT.md` (§2) and are
not repeated here.

Model: `Voucher(code, description, discountPercent, validFrom, validTo,
usageLimit, timesUsed)`. `code` is the key: there is no separate id.

Status: Confirmed — implemented in Phase 8.

## 2. Voucher rules

- `code` is trimmed and stored upper-case; it may contain only letters, digits,
  `-` and `_`, at most 64 characters, and cannot change after creation. Lookups
  ignore case (`sale10` finds `SALE10`).
- A voucher is usable when `validFrom <= now < validTo` and
  `timesUsed < usageLimit`.
- An order uses at most one voucher (FR-VCH-01). The discount is
  `discountPercent` % of the subtotal of the order's non-cancelled lines,
  rounded to 2 decimals (half up). When a line is cancelled later, the discount
  is recalculated the same way on the remaining lines, so the total never goes
  negative.
- Using a voucher at checkout consumes one use with an atomic conditional
  update, so concurrent checkouts cannot exceed `usageLimit` (FR-VCH-04). A use
  is not given back when the order is later cancelled.

`VoucherResponse` `data` shape (admin endpoints and the validity check):

```json
{
  "code": "SALE10",
  "description": "10% off platform campaign",
  "discountPercent": 10,
  "validFrom": "2026-10-01T00:00:00Z",
  "validTo": "2026-11-01T00:00:00Z",
  "usageLimit": 100,
  "timesUsed": 42
}
```

## 3. Customer endpoint

### GET /api/v1/vouchers/{code}/validate

Check that a voucher can be used now, without consuming it. The app calls this
to show the discount before checkout.
Actor: `CUSTOMER` (`Authorization: Bearer <access-token>` required).

Success: `200 OK`, `ApiResponse<VoucherResponse>`.

Errors (common envelope):

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401` | Missing/invalid token. |
| `FORBIDDEN` | `403` | Caller is not `CUSTOMER`. |
| `RESOURCE_NOT_FOUND` | `404` | Unknown `code`. |
| `VOUCHER_EXPIRED` | `409` | Outside `[validFrom, validTo)`. |
| `VOUCHER_EXHAUSTED` | `409` | `timesUsed >= usageLimit`. |

Maps to FR-VCH-02, FR-VCH-03.

## 4. Admin endpoints

Prefix: `/api/v1/admin/vouchers`. Actor: `ADMIN` for all four endpoints;
`FORBIDDEN` (`403`) for any other role.

### GET /api/v1/admin/vouchers

List vouchers, newest `validFrom` first. Query: `page` (default `0`), `size`
(default `20`). Success: `200 OK`, `ApiResponse<PageResponse<VoucherResponse>>`.

### POST /api/v1/admin/vouchers

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

Rules: `code` as in §2 and unique ignoring case; `discountPercent` in
`1..100`; `validFrom < validTo`; `usageLimit >= 1`. `timesUsed` starts at `0`.

Success: `201 Created`, `ApiResponse<VoucherResponse>`.

Errors: `VALIDATION_ERROR` (`400`) for a rule violation or a duplicate `code`.

### PATCH /api/v1/admin/vouchers/{code}

Update a voucher. All fields optional; only present fields change; `code`
cannot change.

```json
{
  "description": "10% off, extended",
  "discountPercent": 15,
  "validFrom": "2026-10-01T00:00:00Z",
  "validTo": "2026-11-15T00:00:00Z",
  "usageLimit": 200
}
```

The result must still satisfy `validFrom < validTo`, and `usageLimit` cannot go
below the current `timesUsed`. Once the voucher has been used (`timesUsed > 0`),
`discountPercent` cannot change: orders that already used it recalculate their
discount with the voucher's percentage when a line is cancelled, so changing it
would change their totals.

Success: `200 OK`, `ApiResponse<VoucherResponse>`.

Errors: `RESOURCE_NOT_FOUND` (`404`) for an unknown `code`;
`VALIDATION_ERROR` (`400`) for a rule violation, including a new
`discountPercent` on a voucher that has already been used.

### DELETE /api/v1/admin/vouchers/{code}

Expire a voucher. Not a physical delete: it sets `validTo = now`, so validation
and checkout reject further uses while history (`timesUsed`, orders that used
it) is kept. For a voucher that has not started yet, `validFrom` also moves to
just before `now` so that `validFrom < validTo` still holds.

Success: `200 OK`, `ApiResponse<VoucherResponse>` with the new dates.

Errors: `RESOURCE_NOT_FOUND` (`404`) for an unknown `code`.

Maps to FR-VCH-02, FR-VCH-03.
