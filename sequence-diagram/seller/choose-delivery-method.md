# Shipment - Choose Delivery Method — Sequence Diagram

- **Use-case spec:** [choose-delivery-method.vi.md](../../usecase-specification/seller/choose-delivery-method.vi.md)
- **API route/status:** `GET /api/v1/seller/carriers, POST /api/v1/seller/shipments (deliveryMethod, carrierId)` — Confirmed
- **Feature API contract:** [features/shipment/API_CONTRACT.md](../../features/shipment/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as SellerShipmentController
    participant Service as CarrierService
    participant Users as UserRepository
    participant Carriers as CarrierRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Choose Delivery Method
    Client->>Security: GET /api/v1/seller/carriers, POST /api/v1/seller/shipments (deliveryMethod, carrierId)
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: listActiveCarriers(sellerId)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Carriers: findByActiveTrueOrderByNameAsc
            Carriers->>DB: SELECT carriers
            Note over Client,Service: The choice is checked by ShipmentService in POST /seller/shipments
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<List<CarrierResponse>>
            Client-->>Seller: Display the result
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Invalid method and carrier pair at shipment creation
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Seller: Display the error
        end
    end
```
