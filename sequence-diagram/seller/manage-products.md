# Catalog - Manage Products — Sequence Diagram

- **Use-case spec:** [manage-products.vi.md](../../usecase-specification/seller/manage-products.vi.md)
- **API route/status:** `GET seller products route TBD; POST/PATCH/DELETE /api/v1/seller/products` — Mixed: product reads TBD + confirmed writes
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

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
    Seller->>Client: Start the manage products use case
    Client->>Security: GET seller products route TBD, POST/PATCH/DELETE /api/v1/seller/products
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute manage products
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to manage products
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Seller product listing route is not defined, CRUD routes are listed in the linked Catalog API contract.
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
