# Order - Process Own Order Items — Sequence Diagram

- **Use-case spec:** [process-own-order-items.vi.md](../../usecase-specification/seller/process-own-order-items.vi.md)
- **API route/status:** `GET /api/v1/seller/order-items; PATCH confirm/cancel` — Predicted
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as OrderController
    participant Service as OrderService
    participant Repo as OrderItemRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Start the process own order items use case
    Client->>Security: GET /api/v1/seller/order-items, PATCH confirm/cancel
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute process own order items
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to process own order items
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Seller order-item endpoints are Predicted in the Order API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: Contract-defined success response for the selected endpoint
        Client-->>Seller: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Display the error
    end
```
