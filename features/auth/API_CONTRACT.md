# Auth API Contract

Scope: customer register/login/refresh entry points. All responses use the `ApiResponse<T>` envelope and common error codes defined in [..\..\API_CONTRACT.md](..\..\API_CONTRACT.md) (§2 conventions, §3 auth).

Conventions: base path `/api/v1`; money as decimal string in `VND`; timestamps ISO-8601 UTC; `expiresIn` is `900` seconds. Passwords are hashed with Spring Security `PasswordEncoder` and never stored or returned in plaintext (FR-AUTH-03).

### POST /api/v1/auth/register

Status: Confirmed (root §3, SRS §5.1).

- Actor/auth: public, no token required.
- Maps to: FR-AUTH-01, FR-AUTH-02, FR-AUTH-03, FR-AUTH-04.
- Business rule: public registration accepts only `CUSTOMER` (default) or `SELLER`. A seller
  application creates `SELLER` with `PENDING` status and requires admin approval; approval/rejection
  triggers a notification email (best-effort, never rolls back the decision). `ADMIN` accounts come
  only from environment-based bootstrap when no admin exists; the client must never send `ADMIN`.
- Request body (JSON):

```json
{
  "email": "customer@example.com",
  "password": "strong-password",
  "name": "Customer",
  "role": "CUSTOMER"
}
```

| Field | Rule |
|---|---|
| `email` | Required, valid email, unique in `users.email` |
| `password` | Required, min length per validation policy (never persisted raw; stored as `users.passwordHash`) |
| `name` | Required, display name (persisted as `users.full_name`) |
| `role` | Optional, `CUSTOMER` (default) or `SELLER`; `ADMIN` is rejected |

- Success: `201 Created`, `ApiResponse<AuthResponse>`. `data` payload:

```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token",
  "expiresIn": 900,
  "user": {
    "id": "user-uuid",
    "email": "customer@example.com",
    "name": "Customer",
    "role": "CUSTOMER"
  }
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing/invalid email, password, or name; duplicate email |
| `INTERNAL_ERROR` | 500 | Unexpected failure |

### POST /api/v1/auth/login

Status: Confirmed (root §3, SRS §5.1).

- Actor/auth: public, no token required.
- Maps to: FR-AUTH-02, FR-AUTH-03, FR-AUTH-04.
- Request body (JSON):

```json
{
  "email": "customer@example.com",
  "password": "strong-password"
}
```

- Success: `200 OK`, `ApiResponse<AuthResponse>` with the same `data` shape as register (`accessToken`, `refreshToken`, `expiresIn: 900`, `user{id, email, name, role}`). `role` reflects the stored `users.role` (`CUSTOMER`, `SELLER`, or `ADMIN`).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Malformed request body |
| `AUTHENTICATION_REQUIRED` | 401 | Wrong email or password, disabled account (`users.enabled = false`) |
| `INTERNAL_ERROR` | 500 | Unexpected failure |

### POST /api/v1/auth/refresh

Status: Confirmed (root §3, SRS §5.1).

- Actor/auth: public endpoint authenticated by a valid refresh token (no access token required).
- Maps to: FR-AUTH-02, FR-AUTH-04.
- Request body (JSON):

```json
{
  "refreshToken": "jwt-refresh-token"
}
```

- Success: `200 OK`, `ApiResponse<AuthResponse>`. Returns a new token pair and revokes the used
  refresh token (rotation); the client must use the new `accessToken` and `refreshToken`. Same `data` shape as above.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing `refreshToken` |
| `AUTHENTICATION_REQUIRED` | 401 | Expired, revoked, or invalid refresh token |
| `INTERNAL_ERROR` | 500 | Unexpected failure |

### POST /api/v1/auth/logout

Status: Confirmed — implemented in Phase 1 (`refresh_tokens` storage from migration V12).

- Actor/auth: authenticated customer/seller/admin (access token) revoking their own refresh token.
- Maps to: FR-AUTH-02, FR-AUTH-04.
- Request body (JSON):

```json
{
  "refreshToken": "jwt-refresh-token"
}
```

- Success: `200 OK`, `ApiResponse<null>` (the envelope omits `data`). The refresh token is revoked;
  reuse returns `401` with `AUTHENTICATION_REQUIRED`.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing `refreshToken` |
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired access token, or revoked refresh token |
| `INTERNAL_ERROR` | 500 | Unexpected failure |
