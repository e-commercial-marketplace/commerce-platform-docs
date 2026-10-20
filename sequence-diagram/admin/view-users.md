# User - View Users — Sequence Diagram

- **Use-case spec:** [view-users.vi.md](../../usecase-specification/admin/view-users.vi.md)
- **API route/status:** `GET /api/v1/admin/users` — Confirmed
- **Feature API contract:** [features/user/API_CONTRACT.md](../../features/user/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

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
    Admin->>Client: Start the view users use case
    Client->>Security: GET /api/v1/admin/users
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute view users
    alt Main flow succeeds
        Service->>Repo: Query UserRepository for permitted view users data
        Repo->>DB: SELECT
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin user endpoints are Confirmed in the User API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
