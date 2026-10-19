# Voucher - Manage Vouchers — Sequence Diagram

- **Use-case spec:** [manage-vouchers.vi.md](../../usecase-specification/admin/manage-vouchers.vi.md)
- **API route/status:** `GET/POST/PATCH/DELETE /api/v1/admin/vouchers` — Predicted
- **Feature API contract:** [features/voucher/API_CONTRACT.md](../../features/voucher/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as VoucherController
    participant Service as VoucherService
    participant Repo as VoucherRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Start the manage vouchers use case
    Client->>Security: GET/POST/PATCH/DELETE /api/v1/admin/vouchers
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute manage vouchers
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to manage vouchers
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin voucher endpoints are Predicted, DELETE expires the voucher rather than physically deleting it.
        Service-->>Controller: Use-case result
        Controller-->>Client: Response defined by each selected endpoint contract
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
