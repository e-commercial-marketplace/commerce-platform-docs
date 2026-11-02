# User - Manage Profile — Sequence Diagram

- **Use-case spec:** [manage-profile.vi.md](../../usecase-specification/customer/manage-profile.vi.md)
- **API route/status:** `GET /api/v1/users/me, PATCH /api/v1/users/me` — Confirmed
- **Feature API contract:** [features/user/API_CONTRACT.md](../../features/user/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as UserController
    participant Service as UserService
    participant Repo as UserRepository
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Manage Profile
    Client->>Security: GET /api/v1/users/me, PATCH /api/v1/users/me
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated authenticated user principal
        Controller->>Service: getProfile, then updateProfile(callerId, request)
        alt Request succeeds
            Service->>Repo: Load the user from the token id
            Repo->>DB: SELECT users
            Service->>Repo: saveAndFlush the changed fields
            Repo->>DB: UPDATE users
            Service->>Cache: Evict the seller shop entry after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<UserProfileResponse>
            Client-->>Customer: Display the result
        else Invalid name, phone or address
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Customer: Display the error
        end
    end
```
