# Catalog - Delete or Unlist Product — Sequence Diagram

- **Use-case spec:** [delete-or-unlist-product.vi.md](../../usecase-specification/seller/delete-or-unlist-product.vi.md)
- **API route/status:** `DELETE /api/v1/seller/products/{id} (soft-delete/unlist)` — Confirmed
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
    Seller->>Client: Start the delete or unlist product use case
    Client->>Security: DELETE /api/v1/seller/products/{id} (soft-delete/unlist)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute delete or unlist product
    alt Main flow succeeds
        Service->>Repo: Apply the contract-defined delete/expiry behavior for delete or unlist product
        Repo->>DB: SELECT and apply the contract-defined delete/expiry update
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: Contract-defined delete/expiry response
        Client-->>Seller: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Display the error
    end
```
