# Auth - Logout — Sequence Diagram

- **Use-case spec:** [logout.vi.md](../../usecase-specification/customer/logout.vi.md)
- **API route/status:** `POST /api/v1/auth/logout` — Confirmed
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AuthController
    participant Service as AuthService
    participant Tokens as RefreshTokenService
    participant Repo as RefreshTokenRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Logout
    Client->>Security: POST /api/v1/auth/logout
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated authenticated user principal
        Controller->>Service: logout(request, callerId)
        alt Request succeeds
            Service->>Tokens: revokeRefreshToken(token, caller)
            Tokens->>Repo: Find the token by SHA-256 hash
            Repo->>DB: SELECT refresh_tokens
            Tokens->>Repo: revokeIfActive
            Repo->>DB: UPDATE refresh_tokens SET revoked_at
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<Void>
            Client-->>Customer: Display the result
        else Unknown, revoked or foreign refresh token
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 401 AUTHENTICATION_REQUIRED
            Client-->>Customer: Display the error
        end
    end
```
