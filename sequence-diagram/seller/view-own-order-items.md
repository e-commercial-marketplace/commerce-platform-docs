# Order - View Own Order Items — Sequence Diagram

- **Use-case spec:** [view-own-order-items.vi.md](../../usecase-specification/seller/view-own-order-items.vi.md)
- **API route/status:** `GET /api/v1/seller/order-items` — Predicted
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
    Seller->>Client: Start the view own order items use case
    Client->>Security: GET /api/v1/seller/order-items
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute view own order items
    alt Main flow succeeds
        Service->>Repo: Query OrderItemRepository for permitted view own order items data
        Repo->>DB: SELECT
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Seller order-item endpoints are Predicted in the Order API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Seller: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Display the error
    end
```
