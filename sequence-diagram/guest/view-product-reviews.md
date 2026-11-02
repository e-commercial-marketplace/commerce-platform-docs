# Review - View Product Reviews — Sequence Diagram

- **Use-case spec:** [view-product-reviews.vi.md](../../usecase-specification/guest/view-product-reviews.vi.md)
- **API route/status:** `GET /api/v1/products/{productId}/reviews` — Confirmed
- **Feature API contract:** [features/review/API_CONTRACT.md](../../features/review/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ReviewController
    participant Service as ReviewService
    participant Products as ProductRepository
    participant Repo as ReviewRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: View Product Reviews
    Client->>Security: GET /api/v1/products/{productId}/reviews
    Security->>Controller: Forward the request
    Controller->>Service: listProductReviews(productId, page, size)
    alt Request succeeds
        Service->>Products: findByIdAndActiveTrue
        Products->>DB: SELECT products
        Service->>Repo: findByProductId, newest first
        Repo->>DB: SELECT reviews
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<PageResponse<ReviewResponse>>
        Client-->>Guest: Display the result
    else Unknown or hidden product
        Service-->>Controller: Propagate the error
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 404 RESOURCE_NOT_FOUND
        Client-->>Guest: Display the error
    end
```
