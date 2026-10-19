# Payment - Approve Refund — Sequence Diagram

- **Use-case spec:** [approve-refund.vi.md](../../usecase-specification/admin/approve-refund.vi.md)
- **API route/status:** `No refund endpoint is defined; route TBD` — TBD
- **Feature API contract:** [features/payment/API_CONTRACT.md](../../features/payment/API_CONTRACT.md)

> **Target sequence:** route and/or persistence contract is not defined yet. Layer names are planned placeholders; the backend scaffold does not yet implement them. Do not treat this as a frozen API.

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as RefundAdminController (TBD)
    participant Service as PaymentService
    participant Repo as PaymentRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Start the approve refund use case
    Client->>Security: Planned operation: approve the refund request (endpoint TBD)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute approve the refund request
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to approve the refund request
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Service,DB: The payment contract has no refund endpoint or VNPAY refund operation, do not infer an external refund call.
        Service-->>Controller: Use-case result
        Controller-->>Client: Success response/status TBD
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
