# Catalog - Browse Catalog — Sequence Diagram

- **Use-case spec:** [browse-catalog.vi.md](../../usecase-specification/customer/browse-catalog.vi.md)
- **API route/status:** `GET /api/v1/products, GET /api/v1/categories` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ProductController
    participant Service as ProductService
    participant Repo as ProductRepository
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Browse Catalog
    Client->>Security: GET /api/v1/products, GET /api/v1/categories
    Security->>Controller: Forward the request
    Controller->>Service: searchListedProducts(request)
    Service->>Cache: Read the 60-second search page
    Service->>Repo: On a miss, query listed products (active = true)
    Repo->>DB: SELECT products with category and seller
    Note over Client,Service: GET /categories goes through CategoryService and the categories cache
    Service-->>Controller: Use-case result
    Controller-->>Client: 200 OK ApiResponse<PageResponse<ProductSummaryResponse>>
    Client-->>Customer: Display the result
```
