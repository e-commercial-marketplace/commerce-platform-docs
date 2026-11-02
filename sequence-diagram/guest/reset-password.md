# Auth - Reset Password — Sequence Diagram

- **Use-case spec:** [reset-password.vi.md](../../usecase-specification/guest/reset-password.vi.md)
- **API route/status:** `POST /api/v1/auth/password/forgot, POST /api/v1/auth/password/reset` — Confirmed
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AuthController
    participant Service as PasswordResetService
    participant Repo as UserRepository
    participant Codes as PasswordResetCodeRepository
    participant Mail as EmailService
    participant Tokens as RefreshTokenService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Reset Password
    Client->>Security: POST /api/v1/auth/password/forgot, POST /api/v1/auth/password/reset
    Security->>Controller: Forward the request
    Controller->>Service: sendResetCode(request), then resetPassword(request)
    alt Request succeeds
        Note over Client,Service: Step 1 is POST /auth/password/forgot, which always answers 200
        Service->>Repo: Find the enabled user by email
        Service->>Codes: Check cooldown and hourly limit
        Service->>Codes: Invalidate old codes and save the new BCrypt hash
        Codes->>DB: UPDATE and INSERT password_reset_codes
        Service->>Mail: Send the code (a mail failure is only logged)
        Note over Client,Service: Step 2 is POST /auth/password/reset
        Service->>Codes: Load the latest unused code and compare
        Service->>Repo: Set the new password hash
        Repo->>DB: UPDATE users
        Service->>Tokens: Revoke all refresh tokens
        Tokens->>DB: UPDATE refresh_tokens
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<Void>
        Client-->>Guest: Display the result
    else Invalid body, or the code is wrong, expired, used or out of attempts
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 400 VALIDATION_ERROR
        Client-->>Guest: Display the error
    end
```
