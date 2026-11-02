# Shipment - Update Shipment Status — Sequence Diagram

- **Use-case spec:** [update-shipment-status.vi.md](../../usecase-specification/seller/update-shipment-status.vi.md)
- **API route/status:** `PATCH /api/v1/seller/shipments/{id}/status` — Confirmed
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
    participant Orders as OrderService
    participant Products as ProductService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Update Shipment Status
    Client->>Security: PATCH /api/v1/seller/shipments/{id}/status
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: updateShipmentStatus(sellerId, id, request)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Repo: Lock the shipment, its lines and the order
            Repo->>DB: SELECT ... FOR UPDATE
            Service->>Repo: Move to the new status
            Repo->>DB: UPDATE shipments
            alt DELIVERED
                Service->>Orders: markDeliveredIfAllShipped(order)
            else RETURNED
                Service->>Products: increaseStock for each line
            else CANCELLED
                Service->>Orders: Cancel the lines, then recalculate totals
                Service->>Products: increaseStock for each line
            else PREPARING, SHIPPED or FAILED
                Note over Service,Repo: Only the status changes
            end
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<ShipmentResponse>
            Client-->>Seller: Display the result
        else Transition not allowed, missing tracking code, or cancelling a paid order's shipment
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Seller: Display the error
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Another seller's shipment
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Unknown shipment
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Seller: Display the error
        end
    end
```
