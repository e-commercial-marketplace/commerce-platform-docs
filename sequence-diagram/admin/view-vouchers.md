# Voucher - View Vouchers — Sequence Diagram

- **Use-case spec:** [view-vouchers.vi.md](../../usecase-specification/admin/view-vouchers.vi.md)
- **API route/status:** `GET /api/v1/admin/vouchers` — Confirmed
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
    Admin->>Client: View Vouchers
    Client->>Security: GET /api/v1/admin/vouchers
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: listVouchers(page, size)
        Service->>Repo: findAll, validFrom descending then code
        Repo->>DB: SELECT vouchers
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<PageResponse<VoucherResponse>>
        Client-->>Admin: Display the result
    end
```
