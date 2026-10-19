# Catalog - Browse Public Catalog — Sequence Diagram

- **Use-case spec:** [browse-public-catalog.vi.md](../../usecase-specification/guest/browse-public-catalog.vi.md)
- **API route/status:** `GET /api/v1/products and GET /api/v1/categories` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ProductController
    participant Service as CatalogService
    participant Repo as ProductRepository
    participant CategoryRepo as CategoryRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Start the browse public catalog use case
    Client->>Security: GET /api/v1/products and GET /api/v1/categories
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute browse public catalog
    alt Main flow succeeds
        Service->>Repo: Query ProductRepository for permitted browse public catalog data
        Repo->>DB: SELECT
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service->>CategoryRepo: Query CategoryRepository for visible categories
        CategoryRepo->>DB: SELECT categories
        DB-->>CategoryRepo: Category rows
        CategoryRepo-->>Service: Category list
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Guest: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Guest: Display the error
    end
```
