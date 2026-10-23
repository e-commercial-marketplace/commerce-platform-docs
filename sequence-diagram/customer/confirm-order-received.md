# Order - Confirm Order Received — Sequence Diagram

- **Use-case spec:** [confirm-order-received.vi.md](../../usecase-specification/customer/confirm-order-received.vi.md)
- **API route/status:** `PATCH /api/v1/orders/{id}/received` — Confirmed
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

> Confirming receipt closes the order (`DELIVERED -> COMPLETED`) and changes no payment: COD was recorded as collected at delivery. Without confirmation, a background job completes the order 3 days after its last shipment was delivered.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as OrderController
    participant Service as OrderService
    participant Repo as OrderRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    participant Job as OrderCompletionJob
    Customer->>Client: Confirm the order was received
    Client->>Security: PATCH /api/v1/orders/{id}/received with customer JWT
    Security->>Controller: Forward authenticated CUSTOMER principal
    Controller->>Service: Confirm receipt for the caller's order
    Service->>Repo: Lock the order and check it belongs to the caller
    Repo->>DB: SELECT ... FOR UPDATE
    DB-->>Repo: Order row
    Repo-->>Service: Locked order
    alt Order is DELIVERED
        Service->>Repo: Mark the order COMPLETED
        Repo->>DB: UPDATE orders SET status = 'COMPLETED'
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<OrderResponse>
        Client-->>Customer: Display the completed order
    else Unknown or another customer's order
        Service-->>Controller: Propagate RESOURCE_NOT_FOUND
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 404 RESOURCE_NOT_FOUND
        Client-->>Customer: Display the error
    else Order is not DELIVERED
        Service-->>Controller: Propagate VALIDATION_ERROR
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 400 VALIDATION_ERROR
        Client-->>Customer: Display the error
    end
    loop Periodically
        Job->>DB: UPDATE orders SET status = 'COMPLETED' WHERE status = 'DELIVERED' and no shipment was delivered in the last 3 days
    end
```
