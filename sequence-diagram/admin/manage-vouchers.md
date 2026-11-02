# Voucher - Manage Vouchers — Sequence Diagram

- **Use-case spec:** [manage-vouchers.vi.md](../../usecase-specification/admin/manage-vouchers.vi.md)
- **API route/status:** `GET/POST/PATCH/DELETE /api/v1/admin/vouchers` — Confirmed
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
    Admin->>Client: Manage Vouchers
    Client->>Security: GET/POST/PATCH/DELETE /api/v1/admin/vouchers
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: Run the chosen voucher operation
        Note over Client,Service: See view-vouchers, create-voucher, update-voucher and expire-voucher
        Service->>Repo: Read or change vouchers
        Repo->>DB: SELECT, INSERT or UPDATE vouchers
        Service-->>Controller: Use-case result
        Controller-->>Client: Response of the chosen operation
        Client-->>Admin: Display the result
    end
```
