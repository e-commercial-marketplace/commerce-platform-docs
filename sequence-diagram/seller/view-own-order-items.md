# Order - View Own Order Items — Sequence Diagram

- **Use-case spec:** [view-own-order-items.vi.md](../../usecase-specification/seller/view-own-order-items.vi.md)
- **API route/status:** `GET /api/v1/seller/order-items?status=&orderId=` — Confirmed
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
    Seller->>Client: View Own Order Items
    Client->>Security: GET /api/v1/seller/order-items?status=&orderId=
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: listMyOrderItems(sellerId, status, orderId, page, size)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Items: findForSeller(seller, status, orderId)
            Items->>DB: SELECT order_items
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<PageResponse<OrderItemResponse>>
            Client-->>Seller: Display the result
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        end
    end
```
