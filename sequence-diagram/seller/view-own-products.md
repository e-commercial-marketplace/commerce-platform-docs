# Catalog - View Own Products — Sequence Diagram

- **Use-case spec:** [view-own-products.vi.md](../../usecase-specification/seller/view-own-products.vi.md)
- **API route/status:** `GET /api/v1/seller/products, GET /api/v1/seller/products/{id}` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as SellerProductController
    participant Service as SellerProductService
    participant Users as UserRepository
    participant Repo as ProductRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: View Own Products
    Client->>Security: GET /api/v1/seller/products, GET /api/v1/seller/products/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: listMyProducts, then getMyProduct
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Repo: Find with soldBy(seller), q and active
            Repo->>DB: SELECT products
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<PageResponse<ProductSummaryResponse>> or ApiResponse<ProductResponse>
            Client-->>Seller: Display the result
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Another seller's product
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Unknown product
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Seller: Display the error
        end
    end
```
