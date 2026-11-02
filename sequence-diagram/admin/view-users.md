# Admin - View Users — Sequence Diagram

- **Use-case spec:** [view-users.vi.md](../../usecase-specification/admin/view-users.vi.md)
- **API route/status:** `GET /api/v1/admin/users?role=&search=, GET /api/v1/admin/users/{id}` — Confirmed
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
    Admin->>Client: View Users
    Client->>Security: GET /api/v1/admin/users?role=&search=, GET /api/v1/admin/users/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: listUsers, then getUser
        alt Request succeeds
            Service->>Repo: Find with hasRole and emailOrNameContains
            Repo->>DB: SELECT users
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<PageResponse<UserResponse>> or ApiResponse<UserResponse>
            Client-->>Admin: Display the result
        else Unknown user
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Admin: Display the error
        end
    end
```
