# Auth - Admin Login — Sequence Diagram

- **Use-case spec:** [login.vi.md](../../usecase-specification/admin/login.vi.md)
- **API route/status:** `POST /api/v1/auth/login` — Confirmed
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AuthController
    participant Service as AuthService
    participant Repo as UserRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Start the login use case
    Client->>Security: POST /api/v1/auth/login
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute login
    alt Main flow succeeds
        Service->>Repo: Query UserRepository for the account by email and verify the password hash
        Repo->>DB: SELECT account credentials and role by email
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service->>Repo: Persist the refresh token hash for the new session
        Repo->>DB: INSERT refresh_tokens
        DB-->>Repo: Persisted token row
        Repo-->>Service: Session issued
        Note over Service,Repo: Auth owns the identity boundary around the users table, so the auth service reads and writes account credentials and role state.
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Guest: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Guest: Display the error
    end
```
