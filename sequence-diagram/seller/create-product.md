# Catalog - Create Product — Sequence Diagram

- **Use-case spec:** [create-product.vi.md](../../usecase-specification/seller/create-product.vi.md)
- **API route/status:** `POST /api/v1/seller/products` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ProductController
    participant Service as CatalogService
    participant Repo as ProductRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Start the create product use case
    Client->>Security: POST /api/v1/seller/products
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute create product
    alt Main flow succeeds
        Service->>Repo: Validate and create records for create product
        Repo->>DB: INSERT (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
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
