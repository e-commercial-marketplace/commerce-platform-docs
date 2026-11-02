# Admin - Create User — Sequence Diagram

- **Use-case spec:** [create-user.vi.md](../../usecase-specification/admin/create-user.vi.md)
- **API route/status:** `POST /api/v1/admin/users` — Confirmed
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
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Create User
    Client->>Security: POST /api/v1/admin/users
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: createUser(request)
        alt Request succeeds
            Service->>Repo: existsByEmailIgnoreCase
            Service->>Repo: saveAndFlush with the BCrypt hash
            Repo->>DB: INSERT users
            Service-->>Controller: Use-case result
            Controller-->>Client: 201 Created ApiResponse<UserResponse>
            Client-->>Admin: Display the result
        else Invalid body or email already used
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Admin: Display the error
        end
    end
```
