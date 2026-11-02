# Payment - Pay for Order (COD) — Sequence Diagram

- **Use-case spec:** [pay-for-order-cod.vi.md](../../usecase-specification/customer/pay-for-order-cod.vi.md)
- **API route/status:** `POST /api/v1/orders (paymentMethod = COD), PATCH /api/v1/seller/shipments/{id}/status` — Confirmed
- **Feature API contract:** [features/payment/API_CONTRACT.md](../../features/payment/API_CONTRACT.md)

> COD has no payment endpoint of its own. It starts at checkout and completes from the seller's shipment update.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Seller
    participant Shipments as ShipmentService
    participant Orders as OrderService
    participant Payments as PaymentService
    participant DB as PostgreSQL
    Customer->>Client: Choose COD at checkout
    Client->>Security: POST /api/v1/orders with paymentMethod COD
    Security->>Orders: Forward the CUSTOMER principal
    Orders->>Payments: createCodPayment(order)
    Payments->>DB: INSERT payments (COD, PENDING)
    Orders-->>Client: 201 Created ApiResponse<OrderResponse>
    Seller->>Security: PATCH /api/v1/seller/shipments/{id}/status with DELIVERED
    Security->>Shipments: Forward the SELLER principal
    Shipments->>DB: Lock the shipment, lines and order, set DELIVERED
    Shipments->>Orders: markDeliveredIfAllShipped(order)
    alt Every non-cancelled line is in a delivered shipment
        Orders->>DB: UPDATE orders SET status = 'DELIVERED'
        Orders->>Payments: markCodCollected(order)
        Payments->>DB: UPDATE payments SET status = 'COMPLETED', paid_at
    else Some lines are not delivered yet
        Note over Orders,Payments: Order and payment stay unchanged
    end
    Client->>Security: GET /api/v1/orders/{id}
    Security-->>Client: 200 OK with the order and payment status
    Client-->>Customer: Display the status
```
