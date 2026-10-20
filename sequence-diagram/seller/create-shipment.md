# Shipment - Create Shipment — Sequence Diagram

- **Use-case spec:** [create-shipment.vi.md](../../usecase-specification/seller/create-shipment.vi.md)
- **API route/status:** `POST /api/v1/seller/shipments` — Predicted
- **Feature API contract:** [features/shipment/API_CONTRACT.md](../../features/shipment/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Seller client
    participant Security as Spring Security
    participant Controller as ShipmentController
    participant Service as ShipmentService
    participant OrderService as OrderService
    participant ItemRepo as OrderItemRepository
    participant ShipmentRepo as ShipmentRepository
    participant LinkRepo as ShipmentOrderItemRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Select confirmed order items and delivery method
    Client->>Security: POST /api/v1/seller/shipments + seller JWT
    Security->>Controller: Forward authenticated SELLER principal
    Controller->>Service: createShipment(orderId, deliveryMethod, itemIds)
    alt Items are confirmed, eligible, and seller-owned
        Service->>OrderService: Validate confirmed items, order, seller ownership
        OrderService->>ItemRepo: Load requested order items
        ItemRepo->>DB: SELECT order_items and ownership/state
        DB-->>ItemRepo: Item state
        ItemRepo-->>OrderService: Owned confirmed items
        OrderService-->>Service: Validation result
        Service->>ShipmentRepo: Persist PENDING shipment
        ShipmentRepo->>DB: INSERT shipment
        Service->>LinkRepo: Link whole order-item lines to shipment
        LinkRepo->>DB: INSERT shipment_order_items in same transaction
        Note over Service,DB: Both inserts commit in the same transaction.
        Service-->>Controller: Created result
        Controller-->>Client: 201 Created ApiResponse<ShipmentResponse>
        Client-->>Seller: Display the created shipment
    else Items are not eligible, not owned, or already in another shipment
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize API error
        Handler-->>Client: Error ApiResponse per contract, nothing is committed
        Client-->>Seller: Show the rejection
    end
    Note over Service,OrderService: Cross-feature calls use the owning Order service, Shipment does not access Order repositories directly.
    Note over Controller,DB: Shipment endpoints are Predicted and outside the first MVP API contract.
```
