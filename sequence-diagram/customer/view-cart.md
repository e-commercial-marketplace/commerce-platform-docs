# Cart - View Cart — Sequence Diagram

- **Use-case spec:** [view-cart.vi.md](../../usecase-specification/customer/view-cart.vi.md)
- **API route/status:** `GET /api/v1/cart` — Confirmed
- **Feature API contract:** [features/cart/API_CONTRACT.md](../../features/cart/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as CartController
    participant Service as CartService
    participant Cache as Redis cache
    participant Repo as CartItemRepository
    participant Products as ProductRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: View Cart
    Client->>Security: GET /api/v1/cart
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: getCart(callerId)
        Service->>Cache: Read the cached lines
        alt Cache hit
            Service->>Products: Load the current products
            Products->>DB: SELECT products
        else Cache miss or Redis down
            Service->>Repo: findByUserId, newest first
            Repo->>DB: SELECT cart_items
            Service->>Cache: Store the lines
        end
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<CartResponse>
        Client-->>Customer: Display the result
    end
```
