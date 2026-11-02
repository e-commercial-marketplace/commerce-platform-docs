# Shipment - View Own Shipments — Sequence Diagram

- **Use-case spec:** [view-own-shipments.vi.md](../../usecase-specification/seller/view-own-shipments.vi.md)
- **API route/status:** `GET /api/v1/seller/shipments?status=&orderId=` — Confirmed
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
    Seller->>Client: View Own Shipments
    Client->>Security: GET /api/v1/seller/shipments?status=&orderId=
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: listMyShipments(sellerId, status, orderId, page, size)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Repo: findForSeller(seller, status, orderId)
            Repo->>DB: SELECT shipments
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<PageResponse<ShipmentResponse>>
            Client-->>Seller: Display the result
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        end
    end
```
