# Auth - Customer Login — Sequence Diagram

- **Use-case spec:** [login.vi.md](../../usecase-specification/customer/login.vi.md)
- **API route/status:** `POST /api/v1/auth/login` — Confirmed
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AuthController
    participant Service as AuthService
    participant Repo as UserRepository
    participant Tokens as RefreshTokenService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Customer Login
    Client->>Security: POST /api/v1/auth/login
    Security->>Controller: Forward the request
    Controller->>Service: login(request)
    alt Request succeeds
        Service->>Repo: Find the user by email
        Repo->>DB: SELECT users
        Service->>Service: Verify the BCrypt hash and enabled flag
        Service->>Tokens: Create a refresh token
        Tokens->>DB: INSERT refresh_tokens (hash only)
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<AuthResponse>
        Client-->>Guest: Display the result
    else Unknown email, wrong password or disabled account
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Guest: Display the error
    else Invalid body
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 400 VALIDATION_ERROR
        Client-->>Guest: Display the error
    end
```
