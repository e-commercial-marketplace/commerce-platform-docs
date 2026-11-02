# Admin - Update User — Sequence Diagram

- **Use-case spec:** [update-user.vi.md](../../usecase-specification/admin/update-user.vi.md)
- **API route/status:** `PATCH /api/v1/admin/users/{id}` — Confirmed
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
    Admin->>Client: Update User
    Client->>Security: PATCH /api/v1/admin/users/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: updateUser(id, request, adminId)
        alt Request succeeds
            Service->>Repo: Load the user
            Repo->>DB: SELECT users
            Service->>Repo: saveAndFlush the changes
            Repo->>DB: UPDATE users
            opt enabled = false
                Service->>Tokens: Revoke all refresh tokens
            end
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<UserResponse>
            Client-->>Admin: Display the result
        else Invalid data, self-lockout or seller status on a non-seller
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
