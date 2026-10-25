# Voucher - Update Voucher — Sequence Diagram

- **Use-case spec:** [update-voucher.vi.md](../../usecase-specification/admin/update-voucher.vi.md)
- **API route/status:** `PATCH /api/v1/admin/vouchers/{code}` — Confirmed
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
    Admin->>Client: Start the update voucher use case
    Client->>Security: PATCH /api/v1/admin/vouchers/{code}
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute update voucher
    alt Main flow succeeds
        Service->>Repo: Load, validate ownership/state, and update for update voucher
        Repo->>DB: SELECT, validate, then UPDATE
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin voucher endpoints are Predicted, DELETE expires the voucher rather than physically deleting it.
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
