# Cart - Add Cart Item — Sequence Diagram

- **Use-case spec:** [add-cart-item.vi.md](../../usecase-specification/customer/add-cart-item.vi.md)
- **API route/status:** `POST /api/v1/cart/items` — Confirmed
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
    Customer->>Client: Start the add cart item use case
    Client->>Security: POST /api/v1/cart/items
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute add a product and quantity to the cart
    alt Main flow succeeds
        Service->>Repo: Validate and create records for add a product and quantity to the cart
        Repo->>DB: INSERT (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK with the updated cart
        Client-->>Customer: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Customer: Display the error
    end
```
