# Shipment - Manage Carriers — Sequence Diagram

- **Use-case spec:** [configure-platform-delivery.vi.md](../../usecase-specification/admin/configure-platform-delivery.vi.md)
- **API route/status:** `GET`/`POST /api/v1/admin/carriers`, `PATCH /api/v1/admin/carriers/{id}` — Confirmed
- **Feature API contract:** [features/shipment/API_CONTRACT.md](../../features/shipment/API_CONTRACT.md)

> Admin manages the carriers that sellers pick for `PLATFORM_DELIVERY`. A carrier is never deleted; deactivating it hides it from sellers.

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AdminCarrierController
    participant Service as CarrierService
    participant Repo as CarrierRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Add, rename, activate or deactivate a carrier
    Client->>Security: POST /api/v1/admin/carriers or PATCH /api/v1/admin/carriers/{id}
    Security->>Controller: Forward authenticated ADMIN request
    Controller->>Service: Create or update the carrier
    alt Main flow succeeds
        Service->>Repo: Check the name is unused (ignoring case), then save the carrier
        Repo->>DB: SELECT by lower(name), then INSERT or UPDATE carriers
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: 201 or 200 ApiResponse<CarrierResponse>
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
