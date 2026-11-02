# Catalog - Manage Products — Sequence Diagram

- **Use-case spec:** [manage-products.vi.md](../../usecase-specification/seller/manage-products.vi.md)
- **API route/status:** `GET/POST/PATCH/DELETE /api/v1/seller/products` — Confirmed
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
    Seller->>Client: Manage Products
    Client->>Security: GET/POST/PATCH/DELETE /api/v1/seller/products
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: Run the chosen product operation
        alt Request succeeds
            Note over Client,Service: See create-product, view-own-products, update-product, unlist-product and update-product-stock
            Service->>Users: getApprovedSeller(token id)
            Users->>DB: SELECT users
            Service->>Repo: Read or change the seller's own products
            Repo->>DB: SELECT, INSERT or UPDATE products
            Service-->>Controller: Use-case result
            Controller-->>Client: Response of the chosen operation
            Client-->>Seller: Display the result
        else Seller not APPROVED (checked in the database)
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        end
    end
```
