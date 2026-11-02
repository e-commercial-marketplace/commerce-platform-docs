# Auth - Change Password — Sequence Diagram

- **Use-case spec:** [change-password.vi.md](../../usecase-specification/customer/change-password.vi.md)
- **API route/status:** `PATCH /api/v1/users/me/password` — Confirmed
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as UserController
    participant Service as AuthService
    participant Repo as UserRepository
    participant Tokens as RefreshTokenService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Change Password
    Client->>Security: PATCH /api/v1/users/me/password
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated authenticated user principal
        Controller->>Service: changePassword(callerId, request)
        alt Request succeeds
            Service->>Repo: Load the caller
            Repo->>DB: SELECT users
            Service->>Service: Verify the current password and that the new one differs
            Service->>Repo: Set the new password hash
            Repo->>DB: UPDATE users
            Service->>Tokens: Revoke all, then create a new refresh token
            Tokens->>DB: UPDATE and INSERT refresh_tokens
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<AuthResponse>
            Client-->>Customer: Display the result
        else Invalid body, wrong current password or unchanged password
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Customer: Display the error
        else Account disabled
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 401 AUTHENTICATION_REQUIRED
            Client-->>Customer: Display the error
        end
    end
```
