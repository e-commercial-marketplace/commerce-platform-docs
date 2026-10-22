# Auth - Logout — Sequence Diagram

- **Use-case spec:** [logout.vi.md](../../usecase-specification/customer/logout.vi.md)
- **API route/status:** `POST /api/v1/auth/logout` — Confirmed
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

> **Shared use case:** Seller and Admin log out through the same endpoint and the same flow; there is no separate diagram for them. Logout revokes only the refresh token. The access token is stateless and stays valid until it expires (at most 15 minutes).

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AuthController
    participant Service as AuthService
    participant Repo as RefreshTokenRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Start the logout use case
    Client->>Security: POST /api/v1/auth/logout with Bearer access token and refreshToken
    alt Access token missing or expired
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Access token valid
        Security->>Controller: Forward authenticated request with the caller id
        Controller->>Service: Execute logout with refreshToken and caller id
        Service->>Repo: Query RefreshTokenRepository by the SHA-256 hash of the token
        Repo->>DB: SELECT refresh_tokens by token_hash
        DB-->>Repo: Token row or none
        Repo-->>Service: Entity or empty
        alt Token exists and belongs to the caller
            Service->>Repo: Revoke the token only if it is still active
            Repo->>DB: UPDATE refresh_tokens SET revoked_at WHERE token_hash AND revoked_at IS NULL
            DB-->>Repo: Rows changed
            Repo-->>Service: 1 row changed
            Note over Service,Repo: The conditional UPDATE is atomic. If another request revoked the token first, 0 rows change and the caller gets the same 401.
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK response defined by the contract
            Client-->>Customer: Display the result
        else Unknown, revoked, or another user's token
            Service-->>Controller: Propagate exception
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 401 AUTHENTICATION_REQUIRED (same response for every case, nothing is leaked)
            Client-->>Customer: Display the error
        end
    end
```
