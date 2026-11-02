# Cart - Update Cart Item — Sequence Diagram

- **Use-case spec:** [update-cart-item.vi.md](../../usecase-specification/customer/update-cart-item.vi.md)
- **API route/status:** `PATCH /api/v1/cart/items/{id}` — Confirmed
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
    participant Repo as CartItemRepository
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Update Cart Item
    Client->>Security: PATCH /api/v1/cart/items/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: changeItemQuantity(callerId, itemId, request)
        alt Request succeeds
            Service->>Users: Lock the customer row
            Users->>DB: SELECT users FOR UPDATE
            Service->>Repo: Find the line and check the owner
            Repo->>DB: SELECT cart_items
            Service->>Repo: Set the quantity
            Repo->>DB: UPDATE cart_items
            Service->>Cache: Evict the cart after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<CartResponse>
            Client-->>Customer: Display the result
        else Invalid quantity
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Customer: Display the error
        else Another customer's line
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Customer: Display the error
        else Unknown line
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 CART_ITEM_NOT_FOUND
            Client-->>Customer: Display the error
        else Hidden product
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 PRODUCT_INACTIVE
            Client-->>Customer: Display the error
        else Quantity above stock
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 INSUFFICIENT_STOCK
            Client-->>Customer: Display the error
        end
    end
```
