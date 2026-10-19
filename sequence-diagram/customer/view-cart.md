# Cart - View Cart — Sequence Diagram

- **Use-case spec:** [view-cart.vi.md](../../usecase-specification/customer/view-cart.vi.md)
- **API route/status:** `GET /api/v1/cart` — Confirmed
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
    Customer->>Client: Start the view cart use case
    Client->>Security: GET /api/v1/cart
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute view cart
    alt Main flow succeeds
        Service->>Repo: Query CartItemRepository for permitted view cart data
        Repo->>DB: SELECT
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Customer: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Customer: Display the error
    end
```
