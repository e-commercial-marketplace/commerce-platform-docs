# Catalog - Search and Filter Products — Sequence Diagram

- **Use-case spec:** [search-and-filter-products.vi.md](../../usecase-specification/guest/search-and-filter-products.vi.md)
- **API route/status:** `GET /api/v1/products?q=&categoryId=&sellerId=&minPrice=&maxPrice=&minRating=&sort=` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ProductController
    participant Service as ProductService
    participant Cache as Redis cache
    participant Repo as ProductRepository
    participant Index as ProductIndexer
    participant Gemini as Embedding provider
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Search and Filter Products
    Client->>Security: GET /api/v1/products?q=&categoryId=&sellerId=&minPrice=&maxPrice=&minRating=&sort=
    Security->>Controller: Forward the request
    Controller->>Service: searchListedProducts(request)
    alt Request succeeds
        Service->>Cache: Read the 60-second search page
        Service->>Repo: LIKE ignoring accents on name, category name and description with the filters
        Repo->>DB: SELECT products
        opt No product contains the keyword
            Service->>Index: findSimilar(q)
            Index->>Gemini: Embed the keyword
            Index->>DB: Nearest rows in product_embeddings
            Service->>Repo: Same filters with id IN the nearest ids
            Repo->>DB: SELECT products
            Note over Service,Index: If embedding is off or fails the list stays empty
        end
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<PageResponse<ProductSummaryResponse>>
        Client-->>Guest: Display the result
    else minPrice above maxPrice
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 400 VALIDATION_ERROR
        Client-->>Guest: Display the error
    end
```
