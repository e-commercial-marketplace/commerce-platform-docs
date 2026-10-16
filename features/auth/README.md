# Auth Feature

## Purpose

Owns authentication entry points and the security boundary for the marketplace.

## MVP boundary

The first mobile MVP uses customer register/login/refresh before server-backed cart checkout. Seller
approval and Admin authorization remain backend capabilities without mobile screens in this milestone.

Public registration creates `CUSTOMER` by default; a seller application (offered on the
account-creation screen) creates a `SELLER` with `PENDING` status. Public registration must never
create an `ADMIN` account. When no admin exists, the first admin is bootstrapped from environment
configuration (never hard-coded credentials).

## Responsibilities

- Register users.
- Authenticate users and issue JWT access/refresh tokens.
- Refresh access tokens with rotation: a successful refresh returns a new pair and revokes the used
  refresh token.
- Revoke refresh tokens before expiry (logout/admin action); revoked tokens are rejected with 401.
- Expire access and refresh tokens per backend configuration.
- Hash passwords with Spring Security `PasswordEncoder`.
- Resolve the authenticated principal for downstream ownership checks.
- Apply role and seller approval rules when seller approval is part of the MVP.
- Send seller-request and seller-approval notification emails on a best-effort basis: email failure
  must not roll back registration or approval (no message broker in the MVP; retry via DB-backed
  outbox or admin resend if needed).

## Planned API

```text
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
```

## Permissions

| Actor | Allowed operation | Constraint |
|---|---|---|
| Unauthenticated client | Register, login, refresh | Auth endpoints are public; guest access is separately limited to public catalog search and the public AI product-search flow. Registration accepts only `CUSTOMER` or `SELLER`; `ADMIN` can only come from bootstrap. |
| Authenticated user | Logout (revoke own refresh token) | Requires access token; the revoked refresh token is rejected on reuse. |
| CUSTOMER | Use customer-authorized features after login | Identity comes from the authenticated token. |
| SELLER | Use seller-authorized features after login | Seller ownership and approval are checked separately. |
| ADMIN | Use admin-authorized features after login | Role comes from the authenticated principal, not request data. |
| Any client | Send a user ID or role in a request | Must not be trusted for authorization. |

## Focused class diagram

```mermaid
classDiagram
    class User {
        UUID id
        String email
        String passwordHash
        Role role
        SellerStatus sellerStatus?
    }

    class Role {
        <<enumeration>>
        CUSTOMER
        SELLER
        ADMIN
    }

    class SellerStatus {
        <<enumeration>>
        PENDING
        APPROVED
        REJECTED
        SUSPENDED
    }

    User --> Role : role
    User --> SellerStatus : optional seller status
```

## Implementation boundary

Auth establishes identity. Feature services must still enforce resource ownership; a `SELLER` role by
itself does not authorize access to another seller's product, order item, or shipment.

## Data model

Auth owns the identity boundary around the `users` table. It does not own product, order, payment, or
shipment rows; those tables reference `users` for ownership and actor checks.

### Table relationships

- `users.id` is referenced by `products.sellerId`, `orders.userId`, and seller/customer participant fields in other features.
- `users.sellerStatus` is an enum value, not a foreign-key relationship to another table.
- `refresh_tokens.userId -> users.id` stores one row per issued refresh token (SHA-256 hash only,
  migration `V12`); rotation revokes the used row, logout/admin action sets `revokedAt`.
- Ownership must be resolved from the authenticated principal before a feature service reads a related row.

### Field meanings

| Table | Field | Meaning |
|---|---|---|
| `users` | `id` | Primary identity used by ownership and audit relationships. |
| `users` | `email` | Unique login identifier. |
| `users` | `passwordHash` | One-way password hash; never store or return the raw password. |
| `users` | `name` | Logical display/full name of the account. The current V1 SQL column is `full_name`. |
| `users` | `role` | Authorization role: `CUSTOMER`, `SELLER`, or `ADMIN`. |
| `users` | `sellerStatus` | Seller onboarding/operation state; nullable for non-sellers. |
| `users` | `enabled` | Whether the account may authenticate; current V1 persistence field. |
| `users` | `createdAt` / `updatedAt` | Account creation and last-update timestamps. |

`phone` and `address` are logical profile fields in the current ERD/class model but are not present in
the V1 migration yet. Do not assume they are persisted until a migration adds them.
