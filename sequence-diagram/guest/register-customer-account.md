# Auth - Register Customer Account — Sequence Diagram

- **Use-case spec:** [register-customer-account.vi.md](../../usecase-specification/guest/register-customer-account.vi.md)
- **API route/status:** `POST /api/v1/auth/register` — Confirmed
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
    Guest->>Client: Start the register customer account use case
    Client->>Security: POST /api/v1/auth/register
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute register customer account
    alt Main flow succeeds
        Service->>Repo: Validate and create records for register customer account
        Repo->>DB: INSERT (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service->>Repo: Persist the refresh token hash for the new session
        Repo->>DB: INSERT refresh_tokens
        DB-->>Repo: Persisted token row
        Repo-->>Service: Session issued
        Note over Service,Repo: Auth owns the identity boundary around the users table, so the auth service reads and writes account credentials and role state.
        Service-->>Controller: Use-case result
        Controller-->>Client: Success response defined by the contract (usually 201 Created)
        Client-->>Guest: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Guest: Display the error
    end
```
