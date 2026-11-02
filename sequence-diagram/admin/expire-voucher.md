# Voucher - Expire Voucher — Sequence Diagram

- **Use-case spec:** [expire-voucher.vi.md](../../usecase-specification/admin/expire-voucher.vi.md)
- **API route/status:** `DELETE /api/v1/admin/vouchers/{code}` — Confirmed
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
    Admin->>Client: Expire Voucher
    Client->>Security: DELETE /api/v1/admin/vouchers/{code}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: expireVoucher(code)
        alt Request succeeds
            Service->>Repo: findById(normalized code)
            Repo->>DB: SELECT vouchers
            Service->>Repo: Set validTo = now
            Repo->>DB: UPDATE vouchers
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<VoucherResponse>
            Client-->>Admin: Display the result
        else Unknown code
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Admin: Display the error
        end
    end
```
