# Order - Cancel Order Item — Sequence Diagram

- **Use-case spec:** [cancel-order-item.vi.md](../../usecase-specification/seller/cancel-order-item.vi.md)
- **API route/status:** `PATCH /api/v1/seller/order-items/{id}/cancel` — Confirmed
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
    participant Orders as OrderService
    participant Products as ProductService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Cancel Order Item
    Client->>Security: PATCH /api/v1/seller/order-items/{id}/cancel
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: cancelOrderItem(sellerId, itemId, reason)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Items: Lock the order item and check the seller
            Items->>DB: SELECT order_items FOR UPDATE
            Service->>Orders: cancelOrderItem(item, reason)
            Service->>Products: increaseStock (clears the product detail cache)
            Products->>DB: UPDATE products
            Service->>Orders: recalculateTotalsAfterCancel(order)
            Orders->>DB: UPDATE orders and payments
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<OrderItemResponse>
            Client-->>Seller: Display the result
        else Already cancelled
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
        else Line already in a shipment
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 CANNOT_CANCEL_AFTER_SHIPMENT
            Client-->>Seller: Display the error
        else Order already paid
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 CANNOT_CANCEL_AFTER_PAYMENT
            Client-->>Seller: Display the error
        end
    end
```
