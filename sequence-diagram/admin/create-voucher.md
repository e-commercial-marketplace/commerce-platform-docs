# Voucher - Create Voucher — Sequence Diagram

- **Use-case spec:** [create-voucher.vi.md](../../usecase-specification/admin/create-voucher.vi.md)
- **API route/status:** `POST /api/v1/admin/vouchers` — Confirmed
- **Feature API contract:** [features/voucher/API_CONTRACT.md](../../features/voucher/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AdminVoucherController
    participant Service as VoucherService
    participant Repo as VoucherRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Create Voucher
    Client->>Security: POST /api/v1/admin/vouchers
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: createVoucher(request)
        alt Request succeeds
            Service->>Repo: existsById(normalized code)
            Repo->>DB: SELECT vouchers
            Service->>DB: persist and flush
            Note over Service,DB: A duplicate key from a parallel request also answers 400
            Service-->>Controller: Use-case result
            Controller-->>Client: 201 Created ApiResponse<VoucherResponse>
            Client-->>Admin: Display the result
        else Invalid data or duplicate code
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Admin: Display the error
        end
    end
```
