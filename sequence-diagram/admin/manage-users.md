# Admin - Manage Users — Sequence Diagram

- **Use-case spec:** [manage-users.vi.md](../../usecase-specification/admin/manage-users.vi.md)
- **API route/status:** `GET/POST/PATCH/DELETE /api/v1/admin/users, PATCH /api/v1/admin/sellers/{id}/approve or /suspend` — Confirmed
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
    Admin->>Client: Manage Users
    Client->>Security: GET/POST/PATCH/DELETE /api/v1/admin/users, PATCH /api/v1/admin/sellers/{id}/approve or /suspend
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: Run the chosen user operation
        Note over Client,Service: See view-users, create-user, update-user, disable-user and approve-or-suspend-seller
        Service->>Repo: Read or change users
        Repo->>DB: SELECT, INSERT or UPDATE users
        Service-->>Controller: Use-case result
        Controller-->>Client: Response of the chosen operation
        Client-->>Admin: Display the result
    end
```
