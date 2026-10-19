# Cart - Remove Cart Item — Sequence Diagram

- **Use-case spec:** [remove-cart-item.vi.md](../../usecase-specification/customer/remove-cart-item.vi.md)
- **API route/status:** `DELETE /api/v1/cart/items/{id}` — Confirmed
- **Feature API contract:** [features/cart/API_CONTRACT.md](../../features/cart/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as CartController
    participant Service as CartService
    participant Repo as CartItemRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Start the remove cart item use case
    Client->>Security: DELETE /api/v1/cart/items/{id}
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute remove cart item
    alt Main flow succeeds
        Service->>Repo: Apply the contract-defined delete/expiry behavior for remove cart item
        Repo->>DB: SELECT and apply the contract-defined delete/expiry update
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: Contract-defined delete/expiry response
        Client-->>Customer: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Customer: Display the error
    end
```
