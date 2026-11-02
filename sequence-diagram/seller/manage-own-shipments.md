# Shipment - Manage Own Shipments — Sequence Diagram

- **Use-case spec:** [manage-own-shipments.vi.md](../../usecase-specification/seller/manage-own-shipments.vi.md)
- **API route/status:** `POST/GET /api/v1/seller/shipments, PATCH /api/v1/seller/shipments/{id}/status` — Confirmed
- **Feature API contract:** [features/shipment/API_CONTRACT.md](../../features/shipment/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as SellerShipmentController
    participant Service as ShipmentService
    participant Users as UserRepository
    participant Repo as ShipmentRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Manage Own Shipments
    Client->>Security: POST/GET /api/v1/seller/shipments, PATCH /api/v1/seller/shipments/{id}/status
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: Run the chosen shipment operation
        alt Request succeeds
            Note over Client,Service: See create-shipment, view-own-shipments and update-shipment-status
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Repo: Read or change the seller's own shipments
            Repo->>DB: SELECT, INSERT or UPDATE shipments
            Service-->>Controller: Use-case result
            Controller-->>Client: Response of the chosen operation
            Client-->>Seller: Display the result
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        end
    end
```
