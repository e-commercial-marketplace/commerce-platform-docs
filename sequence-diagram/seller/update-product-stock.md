# Catalog - Update Product Stock — Sequence Diagram

- **Use-case spec:** [update-product-stock.vi.md](../../usecase-specification/seller/update-product-stock.vi.md)
- **API route/status:** `PATCH /api/v1/seller/products/{id}/stock` — Confirmed
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
    Seller->>Client: Start the update product stock use case
    Client->>Security: PATCH /api/v1/seller/products/{id}/stock
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute update product stock
    alt Main flow succeeds
        Service->>Repo: Load, validate ownership/state, and update for update product stock
        Repo->>DB: SELECT, validate, then UPDATE
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Seller: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Display the error
    end
```
