# Order - Confirm Order Item — Sequence Diagram

- **Use-case spec:** [confirm-order-item.vi.md](../../usecase-specification/seller/confirm-order-item.vi.md)
- **API route/status:** `PATCH /api/v1/seller/order-items/{id}/confirm` — Predicted
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
    Seller->>Client: Start the confirm order item use case
    Client->>Security: PATCH /api/v1/seller/order-items/{id}/confirm
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute confirm order item
    alt Main flow succeeds
        Service->>Repo: Load, validate ownership/state, and update for confirm order item
        Repo->>DB: SELECT, validate, then UPDATE
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
