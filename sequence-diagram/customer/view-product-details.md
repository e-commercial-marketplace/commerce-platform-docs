# Catalog - View Product Details — Sequence Diagram

- **Use-case spec:** [view-product-details.vi.md](../../usecase-specification/customer/view-product-details.vi.md)
- **API route/status:** `GET /api/v1/products/{id}` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ProductController
    participant Service as ProductService
    participant Cache as Redis cache
    participant Repo as ProductRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: View Product Details
    Client->>Security: GET /api/v1/products/{id}
    Security->>Controller: Forward the request
    Controller->>Service: getListedProduct(id)
    alt Request succeeds
        Service->>Cache: Read the product detail entry
        Service->>Repo: On a miss, findByIdAndActiveTrue
        Repo->>DB: SELECT products
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<ProductResponse>
        Client-->>Customer: Display the result
    else Unknown or hidden product
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 404 RESOURCE_NOT_FOUND
        Client-->>Customer: Display the error
    end
```
