# Auth - Register Account — Sequence Diagram

- **Use-case spec:** [register-account.vi.md](../../usecase-specification/guest/register-account.vi.md)
- **API route/status:** `POST /api/v1/auth/register` — Confirmed
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AuthController
    participant Service as AuthService
    participant Users as UserService
    participant Repo as UserRepository
    participant Tokens as RefreshTokenService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Register Account
    Client->>Security: POST /api/v1/auth/register
    Security->>Controller: Forward the request
    Controller->>Service: register(request)
    alt Request succeeds
        Service->>Repo: Check the email is not used
        Repo->>DB: SELECT users by email
        Service->>Users: Build the user (SELLER starts PENDING)
        Service->>Repo: saveAndFlush with the BCrypt hash
        Repo->>DB: INSERT users
        Service->>Tokens: Create a refresh token
        Tokens->>DB: INSERT refresh_tokens (hash only)
        Service-->>Controller: Use-case result
        Controller-->>Client: 201 Created ApiResponse<AuthResponse>
        Client-->>Guest: Display the result
    else Invalid body, role ADMIN or email already used
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 400 VALIDATION_ERROR
        Client-->>Guest: Display the error
    end
```
