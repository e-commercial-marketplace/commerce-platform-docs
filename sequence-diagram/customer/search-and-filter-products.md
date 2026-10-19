# Catalog - Search and Filter Products — Sequence Diagram

- **Use-case spec:** [search-and-filter-products.vi.md](../../usecase-specification/customer/search-and-filter-products.vi.md)
- **API route/status:** `GET /api/v1/products (query/filter parameters per catalog contract)` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ProductController
    participant Service as CatalogService
    participant Repo as ProductRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Start the search and filter products use case
    Client->>Security: GET /api/v1/products (query/filter parameters per catalog contract)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute search and filter products
    alt Main flow succeeds
        Service->>Repo: Query ProductRepository for permitted search and filter products data
        Repo->>DB: SELECT
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Customer: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Customer: Display the error
    end
```
