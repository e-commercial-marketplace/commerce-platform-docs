# Shipment - Manage Own Shipments — Sequence Diagram

- **Use-case spec:** [manage-own-shipments.vi.md](../../usecase-specification/seller/manage-own-shipments.vi.md)
- **API route/status:** `GET /api/v1/seller/shipments; PATCH /api/v1/seller/shipments/{id}/status` — Predicted
- **Feature API contract:** [features/shipment/API_CONTRACT.md](../../features/shipment/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ShipmentController
    participant Service as ShipmentService
    participant Repo as ShipmentRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Start the manage own shipments use case
    Client->>Security: GET /api/v1/seller/shipments, PATCH /api/v1/seller/shipments/{id}/status
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute manage own shipments
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to manage own shipments
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Seller shipment endpoints are Predicted in the Shipment API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: Contract-defined success response for the selected endpoint
        Client-->>Seller: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Display the error
    end
```
