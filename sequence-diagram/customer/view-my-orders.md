# Order - View My Orders — Sequence Diagram

- **Use-case spec:** [view-my-orders.vi.md](../../usecase-specification/customer/view-my-orders.vi.md)
- **API route/status:** `GET /api/v1/orders, GET /api/v1/orders/{id}` — Confirmed
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

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
    Customer->>Client: View My Orders
    Client->>Security: GET /api/v1/orders, GET /api/v1/orders/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: listMyOrders, then getMyOrder
        alt Request succeeds
            Service->>Repo: findByCustomerId, newest first
            Repo->>DB: SELECT orders
            Service->>Repo: findByIdAndCustomerId for the chosen order
            Repo->>DB: SELECT orders with items, shipments and payments
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<PageResponse<OrderSummaryResponse>> or ApiResponse<OrderResponse>
            Client-->>Customer: Display the result
        else Unknown or foreign order
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Customer: Display the error
        end
    end
```
