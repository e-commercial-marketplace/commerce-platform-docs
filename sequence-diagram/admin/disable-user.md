# Admin - Disable User — Sequence Diagram

- **Use-case spec:** [disable-user.vi.md](../../usecase-specification/admin/disable-user.vi.md)
- **API route/status:** `DELETE /api/v1/admin/users/{id}` — Confirmed
- **Feature API contract:** [features/user/API_CONTRACT.md](../../features/user/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AdminUserController
    participant Service as AdminUserService
    participant Repo as UserRepository
    participant Tokens as RefreshTokenService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Disable User
    Client->>Security: DELETE /api/v1/admin/users/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: disableUser(id, adminId)
        alt Request succeeds
            Service->>Repo: Load the user
            Repo->>DB: SELECT users
            Service->>Tokens: Revoke all refresh tokens
            Tokens->>DB: UPDATE refresh_tokens
            Service->>Repo: Save enabled = false
            Repo->>DB: UPDATE users
            Service-->>Controller: Use-case result
            Controller-->>Client: 204 No Content
            Client-->>Admin: Display the result
        else Disabling your own account
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Admin: Display the error
        else Unknown user
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Admin: Display the error
        end
    end
```
