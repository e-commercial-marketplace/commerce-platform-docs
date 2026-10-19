# Admin - Monitor Marketplace Operations — Sequence Diagram

- **Use-case spec:** [monitor-marketplace-operations.vi.md](../../usecase-specification/admin/monitor-marketplace-operations.vi.md)
- **API route/status:** `No monitoring endpoint is defined; route TBD` — TBD
- **API contract:** no feature API contract exists for this operation yet.

> **Target sequence:** route and/or persistence contract is not defined yet. Layer names are planned placeholders; the backend scaffold does not yet implement them. Do not treat this as a frozen API.

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as MarketplaceOperationsController
    participant Service as MarketplaceOperationsService
    participant OrderSvc as OrderService
    participant Repo as OrderRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Start the monitor marketplace operations use case
    Client->>Security: Planned operation: read marketplace operations metrics (endpoint TBD)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute read marketplace operations metrics
    alt Main flow succeeds
        Service->>OrderSvc: Read aggregate operational data (specific metrics TBD)
        OrderSvc->>Repo: Read order-owned aggregate records
        Repo->>DB: SELECT order data as approved by the future contract
        DB-->>Repo: Read-only aggregate data
        Repo-->>OrderSvc: Metrics source data
        OrderSvc-->>Service: Metrics source data
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
