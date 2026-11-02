# Catalog - Update Product Stock — Sequence Diagram

- **Use-case spec:** [update-product-stock.vi.md](../../usecase-specification/seller/update-product-stock.vi.md)
- **API route/status:** `PATCH /api/v1/seller/products/{id}/stock` — Confirmed
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
    Seller->>Client: Update Product Stock
    Client->>Security: PATCH /api/v1/seller/products/{id}/stock
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: updateStock(sellerId, id, request)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Repo: Find the product and check the owner
            Repo->>DB: SELECT products
            Service->>Repo: Set the stock
            Repo->>DB: UPDATE products
            Service->>Cache: Evict the product detail entry after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<ProductResponse>
            Client-->>Seller: Display the result
        else Negative stock
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
