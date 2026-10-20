# Shipment - Update Shipment Status — Sequence Diagram

- **Use-case spec:** [update-shipment-status.vi.md](../../usecase-specification/seller/update-shipment-status.vi.md)
- **API route/status:** `PATCH /api/v1/seller/shipments/{id}/status` — Predicted
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
    participant ShipmentRepo as ShipmentRepository
    participant CatalogService as CatalogService
    participant ProductRepo as ProductRepository
    participant OrderService as OrderService
    participant PaymentService as PaymentService
    participant PaymentRepo as PaymentRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Choose a valid shipment status and required details
    Client->>Security: PATCH /api/v1/seller/shipments/{id}/status + seller JWT
    Security->>Controller: Forward authenticated SELLER principal
    Controller->>Service: updateStatus(shipmentId, status, trackingCode)
    Service->>ShipmentRepo: Load seller-owned shipment and validate transition
    ShipmentRepo->>DB: SELECT shipment and mapped order items
    DB-->>ShipmentRepo: Current shipment state
    ShipmentRepo-->>Service: Owned shipment and items
    alt Transition is valid
        Service->>ShipmentRepo: Persist status and timestamps
        ShipmentRepo->>DB: UPDATE shipment transactionally
        opt Status is RETURNED after warehouse receipt
            Service->>CatalogService: Restore only the shipped quantity once
            CatalogService->>ProductRepo: Restore product stock
            ProductRepo->>DB: UPDATE products.stock
            DB-->>ProductRepo: Updated stock
            CatalogService-->>Service: Stock restored
        end
        opt Final required shipment is DELIVERED
            Service->>OrderService: Recalculate the aggregate order status from the current shipment states
            OrderService-->>Service: Aggregate result
            Service->>PaymentService: Complete COD only when all required deliveries qualify
            PaymentService->>PaymentRepo: Reconcile COD state
            PaymentRepo->>DB: UPDATE payment/order aggregate transactionally
            DB-->>PaymentRepo: Updated payment state
            PaymentService-->>Service: Reconciliation result
        end
        Service-->>Controller: Updated result
        Controller-->>Client: 200 OK ApiResponse<ShipmentResponse>
        Client-->>Seller: Display the new status
    else Ownership or transition rule fails
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize API error
        Handler-->>Client: Error ApiResponse, no invalid transition is committed
        Client-->>Seller: Show the rejection
    end
    Note over Controller,DB: Shipment endpoint is Predicted, FAILED restores no stock, RETURNED restores only after warehouse receipt.
```
