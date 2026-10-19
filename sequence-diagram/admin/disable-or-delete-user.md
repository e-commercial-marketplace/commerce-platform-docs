# User - Disable or Delete User — Sequence Diagram

- **Use-case spec:** [disable-or-delete-user.vi.md](../../usecase-specification/admin/disable-or-delete-user.vi.md)
- **API route/status:** `PATCH /api/v1/admin/users/{id} or DELETE /api/v1/admin/users/{id}` — Predicted
- **Feature API contract:** [features/user/API_CONTRACT.md](../../features/user/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as UserController
    participant Service as UserService
    participant Repo as UserRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Start the disable or delete user use case
    Client->>Security: PATCH /api/v1/admin/users/{id} or DELETE /api/v1/admin/users/{id}
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute disable or delete user
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to disable or delete user
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin user endpoints are Predicted in the User API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: Contract-defined success response for the selected endpoint
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
