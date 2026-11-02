# Order - Process Own Order Items — Sequence Diagram

- **Use-case spec:** [process-own-order-items.vi.md](../../usecase-specification/seller/process-own-order-items.vi.md)
- **API route/status:** `GET /api/v1/seller/order-items, PATCH /api/v1/seller/order-items/{id}/confirm, PATCH /api/v1/seller/order-items/{id}/cancel` — Confirmed
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as SellerOrderController
    participant Service as SellerOrderService
    participant Users as UserRepository
    participant Items as OrderItemRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Process Own Order Items
    Client->>Security: GET /api/v1/seller/order-items, PATCH /api/v1/seller/order-items/{id}/confirm, PATCH /api/v1/seller/order-items/{id}/cancel
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: Run the chosen order-item operation
        alt Request succeeds
            Note over Client,Service: See view-own-order-items, confirm-order-item and cancel-order-item
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Items: Read or change the seller's own lines
            Items->>DB: SELECT or UPDATE order_items
            Service-->>Controller: Use-case result
            Controller-->>Client: Response of the chosen operation
            Client-->>Seller: Display the result
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        end
    end
```
