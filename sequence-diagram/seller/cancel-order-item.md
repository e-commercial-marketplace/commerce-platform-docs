# Order - Cancel Order Item — Sequence Diagram

- **Use-case spec:** [cancel-order-item.vi.md](../../usecase-specification/seller/cancel-order-item.vi.md)
- **API route/status:** `PATCH /api/v1/seller/order-items/{id}/cancel` — Predicted
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Seller client
    participant Security as Spring Security
    participant Controller as OrderController
    participant Service as OrderService
    participant ItemRepo as OrderItemRepository
    participant OrderRepo as OrderRepository
    participant CatalogService as CatalogService
    participant ProductRepo as ProductRepository
    participant PaymentService as PaymentService
    participant PaymentRepo as PaymentRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Open the cancel function for an owned order item
    Client->>Security: PATCH /api/v1/seller/order-items/{id}/cancel + seller JWT
    Security->>Controller: Forward authenticated SELLER principal
    Controller->>Service: cancelOrderItem(itemId, cancelReason, principal)
    Service->>ItemRepo: Load the item with seller ownership and the shipment assignment state
    ItemRepo->>DB: SELECT order_items and shipment linkage
    DB-->>ItemRepo: Item state
    ItemRepo-->>Service: Owned item and eligibility state
    alt Item is cancellable before shipment assignment
        Service->>ItemRepo: Mark the item CANCELLED with the optional reason
        ItemRepo->>DB: UPDATE order_items.status
        DB-->>ItemRepo: Cancelled snapshot
        Service->>CatalogService: Restore the cancelled quantity exactly once
        CatalogService->>ProductRepo: Restore product stock
        ProductRepo->>DB: UPDATE products.stock
        DB-->>ProductRepo: Updated stock
        ProductRepo-->>CatalogService: Stock result
        CatalogService-->>Service: Stock restored
        opt Prepayment is still incomplete
            Service->>OrderRepo: Recalculate Order.totalAmount from remaining non-cancelled snapshots
            OrderRepo->>DB: UPDATE orders.total_amount
            DB-->>OrderRepo: Updated order total
            Service->>PaymentService: Recalculate every still-PENDING payment amount
            PaymentService->>PaymentRepo: Recompute pending payment amounts
            PaymentRepo->>DB: UPDATE payments in the same transaction
            DB-->>PaymentRepo: Updated payment state
            PaymentRepo-->>PaymentService: Payment result
            PaymentService-->>Service: Recalculated amounts
        end
        Service-->>Controller: Cancelled item result
        Controller-->>Client: 200 OK ApiResponse<OrderItemResponse>
        Client-->>Seller: Display the cancelled item
    else Item is not owned, already assigned to a shipment, or paid without a refund policy
        Service-->>Controller: Propagate exception, nothing is committed
        Controller->>Handler: Normalize API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Show the rejection
    end
    Note over Service,DB: Cancelled snapshots stay unchanged for history, and the cancelled quantity is restored exactly once.
```
