# Auth API Contract

Scope: customer register/login/refresh entry points. All responses use the `ApiResponse<T>` envelope and common error codes defined in [..\..\API_CONTRACT.md](..\..\API_CONTRACT.md) (§2 conventions, §3 auth).

Conventions: base path `/api`; money as decimal string in `VND`; timestamps ISO-8601 UTC; `expiresIn` is `900` seconds. Passwords are hashed with Spring Security `PasswordEncoder` and never stored or returned in plaintext (FR-AUTH-03).

### POST /api/auth/register

Status: Confirmed (root §3, SRS §5.1).

- Actor/auth: public, no token required.
- Maps to: FR-AUTH-01, FR-AUTH-02, FR-AUTH-03, FR-AUTH-04.
- Business rule: register creates a `CUSTOMER` role only. The client must not send `role` or `sellerStatus`; any such field is ignored. Seller/Admin accounts are created only via admin flows.
- Request body (JSON):

```json
{
  "email": "customer@example.com",
  "password": "strong-password",
  "name": "Customer"
}
```

| Field | Rule |
|---|---|
| `email` | Required, valid email, unique in `users.email` |
| `password` | Required, min length per validation policy (never persisted raw; stored as `users.passwordHash`) |
| `name` | Required, display name (persisted as `users.full_name`) |

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

### POST /api/auth/login

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

### POST /api/auth/refresh

Status: Confirmed (root §3, SRS §5.1).

- Actor/auth: public endpoint authenticated by a valid refresh token (no access token required).
- Maps to: FR-AUTH-02, FR-AUTH-04.
- Request body (JSON):

```json
{
  "refreshToken": "jwt-refresh-token"
}
```

- Success: `200 OK`, `ApiResponse<AuthResponse>`. Returns a new token pair; the client must use the new `accessToken` and `refreshToken`. Same `data` shape as above.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing `refreshToken` |
| `AUTHENTICATION_REQUIRED` | 401 | Expired, revoked, or invalid refresh token |
| `INTERNAL_ERROR` | 500 | Unexpected failure |
