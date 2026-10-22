# Auth - Guest Login with Google — Sequence Diagram

- **Use-case spec:** [login-with-google.vi.md](../../usecase-specification/guest/login-with-google.vi.md)
- **API route/status:** `POST /api/v1/auth/google` — Planned
- **Feature API contract:** [features/auth/API_CONTRACT.md](../../features/auth/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders. The client obtains the Google ID token itself (Google sign-in screen); the backend only receives the token, verifies it, and never trusts an email sent separately by the client. After verification the flow issues the same token pair as `POST /api/v1/auth/login`.

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Google as Google
    participant Security as Spring Security
    participant Controller as AuthController
    participant Service as AuthService
    participant Repo as UserRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Start the Google login use case
    Client->>Google: Open Google sign-in and ask for consent
    Google-->>Client: ID token (signed JWT)
    Client->>Security: POST /api/v1/auth/google with idToken
    Security->>Controller: Forward public request
    Controller->>Service: Execute loginWithGoogle
    opt Google public keys are not cached
        Service->>Google: Fetch the public signing keys
        Google-->>Service: Public keys
    end
    Service->>Service: Verify signature, issuer, audience (configured client IDs) and expiry
    alt Token valid and Google email verified
        Service->>Repo: Query UserRepository for the account by Google subject
        Repo->>DB: SELECT user by google_sub
        DB-->>Repo: Account row or none
        Repo-->>Service: Entity or empty
        opt No account found by Google subject
            Service->>Repo: Query UserRepository for the account by normalized email
            Repo->>DB: SELECT user by email
            DB-->>Repo: Account row or none
            Repo-->>Service: Entity or empty
            alt Account exists with this email
                Service->>Repo: Link the Google subject to the account
                Repo->>DB: UPDATE users SET google_sub
            else No account with this email
                Service->>Repo: Create a CUSTOMER account linked to the Google subject
                Repo->>DB: INSERT users (role CUSTOMER, random unusable password hash, google_sub)
            end
            DB-->>Repo: Persisted account
            Repo-->>Service: Entity
        end
        alt Account is enabled
            Service->>Repo: Persist the refresh token hash for the new session
            Repo->>DB: INSERT refresh_tokens
            DB-->>Repo: Persisted token row
            Repo-->>Service: Session issued
            Note over Service,Repo: Auth owns the identity boundary around the users table, so the auth service reads and writes account credentials, the Google link and role state. The role of a new account is always CUSTOMER and is never taken from the request.
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK response defined by the contract
            Client-->>Guest: Display the result
        else Account is disabled
            Service-->>Controller: Propagate exception (the transaction rolls back any new link)
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 401 AUTHENTICATION_REQUIRED
            Client-->>Guest: Display the error
        end
    else Token invalid, expired, or email not verified
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 401 AUTHENTICATION_REQUIRED (no detail about which check failed)
        Client-->>Guest: Display the error
    end
```
