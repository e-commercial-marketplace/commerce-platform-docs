# Shipment - View My Shipments — Sequence Diagram

- **Use-case spec:** [view-my-shipments.vi.md](../../usecase-specification/customer/view-my-shipments.vi.md)
- **API route/status:** `GET /api/v1/orders/{id}` (the `shipments` array of the order detail) — Confirmed
- **Feature API contract:** [features/shipment/API_CONTRACT.md](../../features/shipment/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ShipmentController
    participant Service as ShipmentService
    participant Repo as ShipmentRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Start the view my shipments use case
    Client->>Security: GET /api/v1/orders/{id}
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute view my shipments
    alt Main flow succeeds
        Service->>Repo: Query ShipmentRepository for permitted view my shipments data
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
