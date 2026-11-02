# Admin - Approve or Suspend Seller — Sequence Diagram

- **Use-case spec:** [approve-or-suspend-seller.vi.md](../../usecase-specification/admin/approve-or-suspend-seller.vi.md)
- **API route/status:** `PATCH /api/v1/admin/sellers/{id}/approve, PATCH /api/v1/admin/sellers/{id}/suspend` — Confirmed
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
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Approve or Suspend Seller
    Client->>Security: PATCH /api/v1/admin/sellers/{id}/approve, PATCH /api/v1/admin/sellers/{id}/suspend
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: approveSeller(id) or suspendSeller(id)
        alt Request succeeds
            Service->>Repo: Load the seller
            Repo->>DB: SELECT users
            Service->>Repo: saveAndFlush the new status
            Repo->>DB: UPDATE users
            Service->>Cache: Evict the seller shop entry after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<UserResponse>
            Client-->>Admin: Display the result
        else Not a seller or status does not allow it
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
