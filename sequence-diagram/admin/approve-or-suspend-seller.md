# User - Approve or Suspend Seller — Sequence Diagram

- **Use-case spec:** [approve-or-suspend-seller.vi.md](../../usecase-specification/admin/approve-or-suspend-seller.vi.md)
- **API route/status:** `PATCH /api/v1/admin/sellers/{id}/approve or /suspend` — Predicted
- **Feature API contract:** [features/user/API_CONTRACT.md](../../features/user/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Admin client
    participant Security as Spring Security
    participant Controller as UserController
    participant Service as UserService
    participant Repo as UserRepository
    participant DB as PostgreSQL
    participant Email as Email provider (TBD)
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Approve or suspend a seller
    Client->>Security: PATCH /api/v1/admin/sellers/{id}/approve or /suspend + admin JWT
    Security->>Controller: Forward authenticated ADMIN principal
    Controller->>Service: Change seller status
    Service->>Repo: Load seller and validate current status
    Repo->>DB: SELECT users by id and role
    DB-->>Repo: Seller state
    alt Status transition is allowed
        Service->>Repo: Save APPROVED or SUSPENDED state
        Repo->>DB: UPDATE users transactionally
        DB-->>Repo: Updated seller
        opt Notification provider is configured
            Service->>Email: Send seller status notification (best effort)
            Email-->>Service: Delivery result, failure does not roll back status
        end
        Service-->>Controller: Result
        Controller-->>Client: 200 OK ApiResponse<UserResponse>
        Client-->>Admin: Display the decision result
    else Invalid seller or status transition
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize API error
        Handler-->>Client: Error ApiResponse
        Client-->>Admin: Show the error
    end
    Note over Service,Email: Email provider selection is not specified, notification is best-effort per Auth API contract.
```
