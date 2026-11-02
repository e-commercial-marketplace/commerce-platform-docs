# Catalog - View Seller Shop — Sequence Diagram

- **Use-case spec:** [view-seller-shop.vi.md](../../usecase-specification/customer/view-seller-shop.vi.md)
- **API route/status:** `GET /api/v1/sellers/{id}, GET /api/v1/products?sellerId={id}` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as SellerShopController
    participant Service as SellerShopService
    participant Cache as Redis cache
    participant Users as UserRepository
    participant Products as ProductRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: View Seller Shop
    Client->>Security: GET /api/v1/sellers/{id}, GET /api/v1/products?sellerId={id}
    Security->>Controller: Forward the request
    Controller->>Service: getShop(sellerId)
    alt Request succeeds
        Service->>Cache: Read the shop entry
        Service->>Users: On a miss, find the APPROVED seller
        Users->>DB: SELECT users
        Service->>Products: countBySellerIdAndActiveTrue
        Products->>DB: SELECT count(*) FROM products
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<SellerShopResponse>
        Client-->>Customer: Display the result
    else Unknown or unapproved seller
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 404 RESOURCE_NOT_FOUND
        Client-->>Customer: Display the error
    end
```
