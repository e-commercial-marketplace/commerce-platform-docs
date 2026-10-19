# Voucher - Delete or Expire Voucher — Sequence Diagram

- **Use-case spec:** [delete-or-expire-voucher.vi.md](../../usecase-specification/admin/delete-or-expire-voucher.vi.md)
- **API route/status:** `DELETE /api/v1/admin/vouchers/{id} (expires the voucher)` — Predicted
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
    Admin->>Client: Start the delete or expire voucher use case
    Client->>Security: DELETE /api/v1/admin/vouchers/{id} (expires the voucher)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute delete or expire voucher
    alt Main flow succeeds
        Service->>Repo: Apply the contract-defined delete/expiry behavior for delete or expire voucher
        Repo->>DB: SELECT and apply the contract-defined delete/expiry update
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin voucher endpoints are Predicted, DELETE expires the voucher rather than physically deleting it.
        Service-->>Controller: Use-case result
        Controller-->>Client: Contract-defined delete/expiry response
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
