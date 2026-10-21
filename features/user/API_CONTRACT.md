# User API Contract

Scope: the current user's profile, admin user management and seller approval state. All endpoints below belong to Phase 1 (Auth + User) and are **Confirmed**. All responses use the `ApiResponse<T>` envelope and common error codes defined in [..\..\API_CONTRACT.md](..\..\API_CONTRACT.md) (§2).

Conventions: base path `/api/v1`; identifiers are UUID strings; timestamps ISO-8601 UTC. User response fields follow the user feature model: `users{id, email, name (V1 column full_name), phone, address, role, sellerStatus, enabled, createdAt, updatedAt}`. `passwordHash` is never returned. `sellerStatus` (`PENDING`, `APPROVED`, `REJECTED`, `SUSPENDED`) is non-null only when `role = SELLER`; it is `null` for `CUSTOMER` and `ADMIN`.

State rule: a seller registers as `PENDING`. An admin moves the seller to `APPROVED` (approve), `REJECTED` (set through `PATCH /admin/users/{id}`) or, for an approved seller, `SUSPENDED` (suspend). A `SUSPENDED` or `REJECTED` seller can be approved again. Only `APPROVED` sellers may use seller-authorized operations.

Self-protection rule: an admin cannot lock themselves out. `PATCH` and `DELETE` on the admin's own account fail with `VALIDATION_ERROR` when they would disable the account or change its role.

Contact rule (Phase 1 addition): `phone` and `address` are optional (`null` until set). `phone` must be a
Vietnamese mobile number: `0` or `+84` followed by 9 digits, where the first of them is `3`, `5`, `7`, `8`, or `9`
(for example `0912345678` or `+84912345678`). The server stores and returns it as `0xxxxxxxxx`. `address`
is at most 500 characters and is trimmed. In an update, an omitted (`null`) field stays unchanged and an
empty string clears it. A user edits their own contact data with `PATCH /users/me`; an admin can edit any
user's contact data with the admin endpoints.

Disabled accounts: `enabled = false` blocks login and refresh immediately and revokes the user's refresh tokens. An access token that was already issued keeps working until it expires (at most 15 minutes).

### GET /api/v1/users/me

Status: Confirmed (implemented in Phase 1).

- Actor/auth: any authenticated user (`Authorization: Bearer <access-token>`). Identity comes from the token, never from request data.
- Success: `200 OK`, `ApiResponse<UserProfileResponse>`. `data` payload (`phone` and `address` are
  `null` until set; a Phase 1 addition):

```json
{
  "id": "user-uuid",
  "email": "customer@example.com",
  "name": "Customer",
  "role": "CUSTOMER",
  "phone": "0912345678",
  "address": "123 Nguyen Trai, District 1, Ho Chi Minh City"
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing, invalid, or expired access token |
| `RESOURCE_NOT_FOUND` | 404 | The authenticated user no longer exists |

### PATCH /api/v1/users/me

Status: Confirmed (implemented as a Phase 1 addition).

- Actor/auth: any authenticated user. The account comes from the access token, never from request data.
- Maps to: FR-AUTH-11.
- Business rule: updates the caller's own `name`, `phone`, and `address` following the contact rule
  above. `email`, `role`, `sellerStatus`, and `enabled` cannot be changed here.
- Request body (JSON, all fields optional):

```json
{
  "name": "Nguyen Van A",
  "phone": "+84912345678",
  "address": "123 Nguyen Trai, District 1, Ho Chi Minh City"
}
```

| Field | Rule |
|---|---|
| `name` | Optional; if sent, not blank and at most 255 characters |
| `phone` | Optional; Vietnamese mobile number, or `""` to clear |
| `address` | Optional; at most 500 characters, or `""` to clear |

- Success: `200 OK`, `ApiResponse<UserProfileResponse>` (same shape as `GET /users/me`). The phone
  number above is returned as `"0912345678"`.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Blank or too long `name`, phone that is not a Vietnamese mobile number, or `address` longer than 500 characters |
| `AUTHENTICATION_REQUIRED` | 401 | Missing, invalid, or expired access token |
| `RESOURCE_NOT_FOUND` | 404 | The authenticated user no longer exists |

### PATCH /api/v1/users/me/password

Status: Confirmed (implemented as a Phase 1 addition).

- Actor/auth: any authenticated user. The account comes from the access token, never from request data.
- Maps to: FR-AUTH-03, FR-AUTH-08.
- Business rule: `currentPassword` must match the stored hash and `newPassword` must differ from it.
  On success the password hash is replaced, all of the user's refresh tokens are revoked (other devices
  are signed out at their next refresh), and a new token pair is returned so the current device stays
  signed in. A wrong current password returns `400`, not `401`, so the client does not treat it as an
  expired session.
- Request body (JSON):

```json
{
  "currentPassword": "old-password",
  "newPassword": "new-strong-password"
}
```

| Field | Rule |
|---|---|
| `currentPassword` | Required |
| `newPassword` | Required, 8 to 72 characters, different from `currentPassword` |

- Success: `200 OK`, `ApiResponse<AuthResponse>` (same shape as login).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Invalid body, wrong current password, or new password equal to the current one |
| `AUTHENTICATION_REQUIRED` | 401 | Missing, invalid, or expired access token, or the account is disabled |
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
      "phone": "0912345678",
      "address": "45 Le Loi, District 1, Ho Chi Minh City",
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
  "phone": null,
  "address": null,
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
- Business rule: an admin may create any role (`CUSTOMER`, `SELLER`, `ADMIN`). A created `SELLER` always starts as `PENDING`; an admin approves them afterwards. Non-sellers have `sellerStatus: null`. Password is hashed server-side into `users.passwordHash`. `enabled` defaults to `true` when omitted. `phone` and `address` are optional and follow the contact rule.
- Request body (JSON):

```json
{
  "email": "staff@example.com",
  "password": "strong-password",
  "name": "Staff",
  "phone": "0987654321",
  "address": "10 Hai Ba Trung, Hanoi",
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
| `VALIDATION_ERROR` | 400 | Invalid email/password/name/role/phone/address; duplicate email; `sellerStatus` set for non-seller |

### PATCH /api/v1/admin/users/{id}

Status: Confirmed (implemented in Phase 1).

- Actor/auth: `ADMIN` only.
- Maps to: FR-AUTH-01.
- Business rule: admin may change `name`, `phone`, `address`, `role`, `sellerStatus` and `enabled`; omitted fields stay unchanged (`phone` and `address` follow the contact rule). Changing `role` away from `SELLER` clears `sellerStatus` to `null`; changing to `SELLER` sets `sellerStatus` to the given value or `PENDING` when omitted. `sellerStatus` can only be set on a `SELLER`. This is also how an admin rejects a seller (`"sellerStatus": "REJECTED"`). Disabling or demoting the admin's own account is rejected (self-protection rule).
- Request body (JSON, all fields optional):

```json
{
  "name": "Updated Name",
  "phone": "0912345678",
  "address": "",
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
| `VALIDATION_ERROR` | 400 | Invalid role/status value, invalid phone or address, `sellerStatus` given for a non-seller, or the admin disables/demotes themselves |
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
