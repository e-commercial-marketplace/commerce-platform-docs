# Order - Confirm Order Item — Sequence Diagram

- **Use-case spec:** [confirm-order-item.vi.md](../../usecase-specification/seller/confirm-order-item.vi.md)
- **API route/status:** `PATCH /api/v1/seller/order-items/{id}/confirm` — Confirmed
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
    participant Payments as PaymentService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Confirm Order Item
    Client->>Security: PATCH /api/v1/seller/order-items/{id}/confirm
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: confirmOrderItem(sellerId, itemId)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Items: Lock the order item and check the seller
            Items->>DB: SELECT order_items FOR UPDATE
            Service->>Payments: isPaidByVnpay(order)
            Service->>Items: Set CONFIRMED
            Items->>DB: UPDATE order_items
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<OrderItemResponse>
            Client-->>Seller: Display the result
        else Line not PENDING, or VNPAY order not paid
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Seller: Display the error
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Another seller's line
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Unknown line
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Seller: Display the error
        end
    end
```
