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

### POST /api/v1/auth/google

Status: Planned (SRS §5.1, FR-AUTH-13).

- Actor/auth: public, no token required. The client first obtains a Google ID token from the Google
  sign-in screen and sends only that token.
- Maps to: FR-AUTH-13, FR-AUTH-04.
- Business rule: the backend verifies the ID token (signature, issuer, audience from the configured
  client IDs, expiry) and requires `email_verified = true`. It never trusts an email sent by the
  client. The account is found by the Google subject, then by normalized email; an existing account
  with the same email is linked to the Google subject, otherwise a `CUSTOMER` account is created. The
  role is never taken from the request. A disabled account is rejected.
- Request body (JSON):

```json
{
  "idToken": "google-id-token"
}
```

- Success: `200 OK`, `ApiResponse<AuthResponse>` with the same `data` shape as login. The status is
  `200` for new and existing accounts alike, so the response does not reveal whether the account
  already existed.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing `idToken` |
| `AUTHENTICATION_REQUIRED` | 401 | Invalid, expired, or wrong-audience token; email not verified by Google; disabled account |
| `INTERNAL_ERROR` | 500 | Unexpected failure |

## Password management (Phase 1 addition)

The endpoints below were added to Phase 1 and are implemented. They share these rules:

- The reset code has 6 digits, is valid for 15 minutes, allows at most 5 wrong attempts, and can be
  used once. Only a hash of the code is stored (`password_reset_codes`, migration V14); the raw code
  exists only in the email.
- A new code replaces any earlier unused code of the same user. A user can request a new code at most
  once per 60 seconds and at most 5 times per hour; requests above these limits are ignored silently.
- Disabled accounts (`users.enabled = false`) never receive a code and cannot reset their password.
- A successful reset or change revokes all of the user's refresh tokens. An access token that was
  already issued keeps working until it expires (at most 15 minutes).
- The new password follows the register rule (8 to 72 characters).
- Email is sent through the provider in SRS §8.5. When email is turned off for local development,
  the code is written to the application log instead.

### POST /api/v1/auth/password/forgot

Status: Confirmed (implemented as a Phase 1 addition).

- Actor/auth: public, no token required.
- Maps to: FR-AUTH-09, FR-AUTH-10.
- Business rule: if the email belongs to an enabled account and the limits above allow it, create a
  new code and email it. The response is the same in every case, so the client cannot tell whether the
  email is registered. An email failure is logged on the server and does not change the response.
- Request body (JSON):

```json
{
  "email": "customer@example.com"
}
```

- Success: `200 OK`, `ApiResponse<null>` (the envelope omits `data`).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing or malformed `email` |
| `INTERNAL_ERROR` | 500 | Unexpected failure |

### POST /api/v1/auth/password/reset

Status: Confirmed (implemented as a Phase 1 addition).

- Actor/auth: public, no token required. The code proves ownership of the email.
- Maps to: FR-AUTH-03, FR-AUTH-09.
- Business rule: checks the latest unused code of the account. A wrong code counts as one attempt;
  after 5 wrong attempts the code stops working. On success the password hash is replaced, the code is
  marked used, and all refresh tokens are revoked. The client then logs in with the new password.
- Request body (JSON):

```json
{
  "email": "customer@example.com",
  "code": "482915",
  "newPassword": "new-strong-password"
}
```

| Field | Rule |
|---|---|
| `email` | Required, valid email |
| `code` | Required, exactly 6 digits |
| `newPassword` | Required, 8 to 72 characters |

- Success: `200 OK`, `ApiResponse<null>`.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Invalid body; or the code is wrong, expired, already used, out of attempts, or the email has no usable code. These cases share one message so the client cannot tell them apart. |
| `INTERNAL_ERROR` | 500 | Unexpected failure |
