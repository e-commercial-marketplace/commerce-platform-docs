# Payment - Pay for Order (COD) — Sequence Diagram

- **Use-case spec:** [pay-for-order-cod.vi.md](../../usecase-specification/customer/pay-for-order-cod.vi.md)
- **API route/status:** `No customer COD endpoint is defined; delivery-result trigger is TBD` — TBD
- **Feature API contract:** [features/shipment/API_CONTRACT.md](../../features/shipment/API_CONTRACT.md)

> **Endpoint gap:** COD cash is offline and the API contract defines settlement from shipment delivery state; it does not define a Customer payment endpoint or delivery-event source.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Handoff as Offline delivery handoff (source TBD)
    participant Controller as ShipmentController (planned)
    participant Service as ShipmentService
    participant ShipmentRepo as ShipmentRepository
    participant PaymentService as PaymentService
    participant PaymentRepo as PaymentRepository
    participant DB as PostgreSQL
    Customer->>Handoff: Receive package and pay cash-on-delivery
    Handoff-->>Controller: Report delivery outcome (API/event source TBD)
    Controller->>Service: Record shipment delivery result
    Service->>ShipmentRepo: Update the shipment result
    ShipmentRepo->>DB: UPDATE shipment status
    DB-->>ShipmentRepo: Updated shipment and order associations
    alt All required non-cancelled shipments are delivered
        Service->>PaymentService: Complete COD for the order
        PaymentService->>PaymentRepo: Update COD payment state
        PaymentRepo->>DB: UPDATE payment/order state transactionally
        DB-->>PaymentRepo: Completed payment state
        PaymentService-->>Service: COD completed
    else More required deliveries remain
        Service->>PaymentService: Keep COD pending
        PaymentService->>PaymentRepo: Confirm pending state
        PaymentRepo->>DB: Read/update payment state
        DB-->>PaymentRepo: Pending payment state
        PaymentService-->>Service: No final settlement yet
    end
    Service-->>Controller: Result
    Controller-->>Handoff: Record/reporting acknowledgement (response contract TBD)
    Note over Customer,Controller: COD cash is offline. The current API contract defines completion from shipment delivery state, not a Customer payment endpoint.
```
