# Shipment - Choose Delivery Method — Sequence Diagram

- **Use-case spec:** [choose-delivery-method.vi.md](../../usecase-specification/seller/choose-delivery-method.vi.md)
- **API route/status:** `POST /api/v1/seller/shipments (deliveryMethod is part of the request)` — Predicted
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
    Seller->>Client: Start the choose delivery method use case
    Client->>Security: POST /api/v1/seller/shipments (deliveryMethod is part of the request)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute choose delivery method
    alt Main flow succeeds
        Service->>Repo: Validate and create records for choose delivery method
        Repo->>DB: INSERT (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Seller shipment endpoints are Predicted in the Shipment API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: Success response defined by the contract (usually 201 Created)
        Client-->>Seller: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Display the error
    end
```
