# User - Create User — Sequence Diagram

- **Use-case spec:** [create-user.vi.md](../../usecase-specification/admin/create-user.vi.md)
- **API route/status:** `POST /api/v1/admin/users` — Confirmed
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
    Admin->>Client: Start the create user use case
    Client->>Security: POST /api/v1/admin/users
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute create user
    alt Main flow succeeds
        Service->>Repo: Validate and create records for create user
        Repo->>DB: INSERT (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin user endpoints are Confirmed in the User API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: Success response defined by the contract (usually 201 Created)
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
