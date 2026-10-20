# User API Contract

Scope: the current user's profile, admin user management and seller approval state. All endpoints below belong to Phase 1 (Auth + User) and are **Confirmed**. All responses use the `ApiResponse<T>` envelope and common error codes defined in [..\..\API_CONTRACT.md](..\..\API_CONTRACT.md) (§2).

Conventions: base path `/api/v1`; identifiers are UUID strings; timestamps ISO-8601 UTC. User response fields follow the user feature model: `users{id, email, name (V1 column full_name), role, sellerStatus, enabled, createdAt, updatedAt}`. `passwordHash` is never returned. `sellerStatus` (`PENDING`, `APPROVED`, `REJECTED`, `SUSPENDED`) is non-null only when `role = SELLER`; it is `null` for `CUSTOMER` and `ADMIN`.

State rule: a seller registers as `PENDING`. An admin moves the seller to `APPROVED` (approve), `REJECTED` (set through `PATCH /admin/users/{id}`) or, for an approved seller, `SUSPENDED` (suspend). A `SUSPENDED` or `REJECTED` seller can be approved again. Only `APPROVED` sellers may use seller-authorized operations.

Self-protection rule: an admin cannot lock themselves out. `PATCH` and `DELETE` on the admin's own account fail with `VALIDATION_ERROR` when they would disable the account or change its role.

Disabled accounts: `enabled = false` blocks login and refresh immediately and revokes the user's refresh tokens. An access token that was already issued keeps working until it expires (at most 15 minutes).

### GET /api/v1/users/me

Status: Confirmed (implemented in Phase 1).

- Actor/auth: any authenticated user (`Authorization: Bearer <access-token>`). Identity comes from the token, never from request data.
- Success: `200 OK`, `ApiResponse<UserSummaryResponse>`. `data` payload:

```json
{
  "id": "user-uuid",
  "email": "customer@example.com",
  "name": "Customer",
  "role": "CUSTOMER"
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing, invalid, or expired access token |
| `RESOURCE_NOT_FOUND` | 404 | The authenticated user no longer exists |

### GET /api/v1/admin/users

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only, `Authorization: Bearer <access-token>` required.
- Maps to: FR-AUTH-01, FR-AUTH-05.
- Query params:

| Parameter | Meaning |
|---|---|
| `page` | Zero-based page index, default `0` |
| `size` | Page size, default `20` |
| `role` | Optional filter: `CUSTOMER`, `SELLER`, or `ADMIN` |
| `search` | Optional free text matched against `email` and `name` |

- Success: `200 OK`, `ApiResponse<PageResponse<UserResponse>>`. `data` payload:

```json
{
  "content": [
    {
      "id": "user-uuid",
      "email": "seller@example.com",
      "name": "Seller",
      "role": "SELLER",
      "sellerStatus": "PENDING",
      "enabled": true,
      "createdAt": "2026-10-12T10:30:00Z",
      "updatedAt": "2026-10-12T10:30:00Z"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Authenticated actor is not `ADMIN` |
| `VALIDATION_ERROR` | 400 | Invalid `page`, `size`, or `role` value |

### GET /api/v1/admin/users/{id}

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only.
- Maps to: FR-AUTH-01.
- Params: `id` (path, UUID of `users.id`).
- Success: `200 OK`, `ApiResponse<UserResponse>`. `data` payload:

```json
{
  "id": "user-uuid",
  "email": "customer@example.com",
  "name": "Customer",
  "role": "CUSTOMER",
  "sellerStatus": null,
  "enabled": true,
  "createdAt": "2026-10-12T10:30:00Z",
  "updatedAt": "2026-10-12T10:30:00Z"
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `RESOURCE_NOT_FOUND` | 404 | No user with `id` |

### POST /api/v1/admin/users

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only.
- Maps to: FR-AUTH-01, FR-AUTH-03.
- Business rule: an admin may create any role (`CUSTOMER`, `SELLER`, `ADMIN`). A created `SELLER` always starts as `PENDING`; an admin approves them afterwards. Non-sellers have `sellerStatus: null`. Password is hashed server-side into `users.passwordHash`. `enabled` defaults to `true` when omitted.
- Request body (JSON):

```json
{
  "email": "staff@example.com",
  "password": "strong-password",
  "name": "Staff",
  "role": "ADMIN",
  "enabled": true
}
```

- Success: `201 Created`, `ApiResponse<UserResponse>` (same shape as GET detail; never includes the password).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `VALIDATION_ERROR` | 400 | Invalid email/password/name/role; duplicate email; `sellerStatus` set for non-seller |

### PATCH /api/v1/admin/users/{id}

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only.
- Maps to: FR-AUTH-01.
- Business rule: admin may change `name`, `role`, `sellerStatus` and `enabled`; omitted fields stay unchanged. Changing `role` away from `SELLER` clears `sellerStatus` to `null`; changing to `SELLER` sets `sellerStatus` to the given value or `PENDING` when omitted. `sellerStatus` can only be set on a `SELLER`. This is also how an admin rejects a seller (`"sellerStatus": "REJECTED"`). Disabling or demoting the admin's own account is rejected (self-protection rule).
- Request body (JSON, all fields optional):

```json
{
  "name": "Updated Name",
  "role": "SELLER",
  "sellerStatus": "PENDING",
  "enabled": false
}
```

- Success: `200 OK`, `ApiResponse<UserResponse>` with the updated user.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `VALIDATION_ERROR` | 400 | Invalid role/status value, `sellerStatus` given for a non-seller, or the admin disables/demotes themselves |
| `RESOURCE_NOT_FOUND` | 404 | No user with `id` |

### DELETE /api/v1/admin/users/{id}

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only.
- Maps to: FR-AUTH-01.
- Business rule: disables the account (`enabled = false`) and revokes its refresh tokens; the row is kept so products, orders and reviews that reference the user stay valid. Repeating the call on a disabled user is allowed. Re-enable with `PATCH` and `"enabled": true`. The admin cannot disable themselves.
- Success: `204 No Content` (no body).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `VALIDATION_ERROR` | 400 | The admin tries to disable their own account |
| `RESOURCE_NOT_FOUND` | 404 | No user with `id` |

### PATCH /api/v1/admin/sellers/{id}/approve

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only.
- Maps to: FR-AUTH-05.
- Business rule: transitions a seller to `APPROVED` from `PENDING`, `REJECTED` or `SUSPENDED` (the last one is the re-approval path). Target must have `role = SELLER`. Only `APPROVED` sellers may manage own products/stock.
- Params: `id` (path, UUID of the seller's `users.id`). Empty request body.
- Success: `200 OK`, `ApiResponse<UserResponse>`. `data` payload shows `"role": "SELLER"`, `"sellerStatus": "APPROVED"`.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `RESOURCE_NOT_FOUND` | 404 | No seller user with `id` |
| `VALIDATION_ERROR` | 400 | Target is not a `SELLER`, or is already `APPROVED` |

### PATCH /api/v1/admin/sellers/{id}/suspend

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only.
- Maps to: FR-AUTH-05.
- Business rule: transitions an `APPROVED` seller to `SUSPENDED`. A suspended seller cannot manage products until re-approved.
- Params: `id` (path, UUID of the seller's `users.id`). Empty request body.
- Success: `200 OK`, `ApiResponse<UserResponse>`. `data` payload shows `"role": "SELLER"`, `"sellerStatus": "SUSPENDED"`.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `RESOURCE_NOT_FOUND` | 404 | No seller user with `id` |
| `VALIDATION_ERROR` | 400 | Target is not a `SELLER`, or current `sellerStatus` is not `APPROVED` |
