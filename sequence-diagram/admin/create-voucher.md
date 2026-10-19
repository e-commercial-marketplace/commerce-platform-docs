# Voucher - Create Voucher — Sequence Diagram

- **Use-case spec:** [create-voucher.vi.md](../../usecase-specification/admin/create-voucher.vi.md)
- **API route/status:** `POST /api/v1/admin/vouchers` — Predicted
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
    Admin->>Client: Start the create voucher use case
    Client->>Security: POST /api/v1/admin/vouchers
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute create voucher
    alt Main flow succeeds
        Service->>Repo: Validate and create records for create voucher
        Repo->>DB: INSERT (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin voucher endpoints are Predicted, DELETE expires the voucher rather than physically deleting it.
        Service-->>Controller: Use-case result
        Controller-->>Client: Success response defined by the contract (usually 201 Created)
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
