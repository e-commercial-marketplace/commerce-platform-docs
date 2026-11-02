# Catalog - Update Product — Sequence Diagram

- **Use-case spec:** [update-product.vi.md](../../usecase-specification/seller/update-product.vi.md)
- **API route/status:** `PATCH /api/v1/seller/products/{id}` — Confirmed
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
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Update Product
    Client->>Security: PATCH /api/v1/seller/products/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: updateProduct(sellerId, id, request)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Repo: Find the product and check the owner
            Repo->>DB: SELECT products
            Service->>Repo: Change the sent fields
            Repo->>DB: UPDATE products
            Service->>Cache: Evict the shop and product detail entries after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<ProductResponse>
            Client-->>Seller: Display the result
        else Invalid data, unknown category, listed product without image, or changed image URL not allowed
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Seller: Display the error
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
