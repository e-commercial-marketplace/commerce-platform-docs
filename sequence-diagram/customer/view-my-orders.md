# Order - View My Orders — Sequence Diagram

- **Use-case spec:** [view-my-orders.vi.md](../../usecase-specification/customer/view-my-orders.vi.md)
- **API route/status:** `GET /api/v1/orders and GET /api/v1/orders/{id}` — Confirmed
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

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
    Customer->>Client: Start the view my orders use case
    Client->>Security: GET /api/v1/orders and GET /api/v1/orders/{id}
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute view my orders
    alt Main flow succeeds
        Service->>Repo: Query OrderRepository for permitted view my orders data
        Repo->>DB: SELECT
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Customer: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Customer: Display the error
    end
```
