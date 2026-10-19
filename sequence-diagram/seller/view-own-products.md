# Catalog - View Own Products — Sequence Diagram

- **Use-case spec:** [view-own-products.vi.md](../../usecase-specification/seller/view-own-products.vi.md)
- **API route/status:** `No seller product-list endpoint is defined; route TBD` — TBD
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

> **Target sequence:** route and/or persistence contract is not defined yet. Layer names are planned placeholders; the backend scaffold does not yet implement them. Do not treat this as a frozen API.

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
    Seller->>Client: Start the view own products use case
    Client->>Security: Planned operation: view own products (endpoint TBD)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute view own products
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to view own products
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: The Catalog API contract does not define a seller-owned product-list endpoint yet.
        Service-->>Controller: Use-case result
        Controller-->>Client: Success response/status TBD
        Client-->>Seller: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Seller: Display the error
    end
```
