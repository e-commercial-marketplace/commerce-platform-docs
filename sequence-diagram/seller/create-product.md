# Catalog - Create Product — Sequence Diagram

- **Use-case spec:** [create-product.vi.md](../../usecase-specification/seller/create-product.vi.md)
- **API route/status:** `POST /api/v1/seller/products` — Confirmed
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
    participant Categories as CategoryRepository
    participant Repo as ProductRepository
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Create Product
    Client->>Security: POST /api/v1/seller/products
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: createProduct(sellerId, request)
        alt Request succeeds
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Categories: Find the category
            Categories->>DB: SELECT categories
            Service->>Repo: Save the product
            Repo->>DB: INSERT products
            Service->>Cache: Evict the seller shop entry after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 201 Created ApiResponse<ProductResponse>
            Client-->>Seller: Display the result
        else Invalid body, unknown category or listed product without image
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Seller: Display the error
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        end
    end
```
