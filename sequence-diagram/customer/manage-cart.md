# Cart - Manage Cart — Sequence Diagram

- **Use-case spec:** [manage-cart.vi.md](../../usecase-specification/customer/manage-cart.vi.md)
- **API route/status:** `GET /api/v1/cart, POST/PATCH/DELETE /api/v1/cart/items` — Confirmed
- **Feature API contract:** [features/cart/API_CONTRACT.md](../../features/cart/API_CONTRACT.md)

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
    Customer->>Client: Manage Cart
    Client->>Security: GET /api/v1/cart, POST/PATCH/DELETE /api/v1/cart/items
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: Run the chosen cart operation
        alt Request succeeds
            Note over Client,Service: See view-cart, add-cart-item, update-cart-item and remove-cart-item for each operation
            Service->>Repo: Read or change the caller's cart lines
            Repo->>DB: SELECT, INSERT, UPDATE or DELETE cart_items
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<CartResponse>
            Client-->>Customer: Display the result
        else Invalid quantity
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Customer: Display the error
        else Not enough stock
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 INSUFFICIENT_STOCK
            Client-->>Customer: Display the error
        end
    end
```
