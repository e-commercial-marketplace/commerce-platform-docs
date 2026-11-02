# Cart - Add Cart Item — Sequence Diagram

- **Use-case spec:** [add-cart-item.vi.md](../../usecase-specification/customer/add-cart-item.vi.md)
- **API route/status:** `POST /api/v1/cart/items` — Confirmed
- **Feature API contract:** [features/cart/API_CONTRACT.md](../../features/cart/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as CartController
    participant Service as CartService
    participant Users as UserRepository
    participant Products as ProductRepository
    participant Repo as CartItemRepository
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Add Cart Item
    Client->>Security: POST /api/v1/cart/items
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: addToCart(callerId, request)
        alt Request succeeds
            Service->>Users: Lock the customer row (serializes this cart)
            Users->>DB: SELECT users FOR UPDATE
            Service->>Products: Find the product and check it is listed
            Products->>DB: SELECT products
            Service->>Repo: Save or increase the line
            Repo->>DB: INSERT or UPDATE cart_items
            Service->>Cache: Evict the cart after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<CartResponse>
            Client-->>Customer: Display the result
        else Invalid body
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Customer: Display the error
        else Unknown product
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Customer: Display the error
        else Hidden product
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 PRODUCT_INACTIVE
            Client-->>Customer: Display the error
        else Total above stock
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 INSUFFICIENT_STOCK
            Client-->>Customer: Display the error
        end
    end
```
