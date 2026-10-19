# Review - Create Product Review — Sequence Diagram

- **Use-case spec:** [create-product-review.vi.md](../../usecase-specification/customer/create-product-review.vi.md)
- **API route/status:** `POST /api/v1/products/{productId}/reviews` — Confirmed
- **Feature API contract:** [features/review/API_CONTRACT.md](../../features/review/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Customer client
    participant Security as Spring Security
    participant Controller as ReviewController
    participant Service as ReviewService
    participant Repo as ReviewRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Open the purchased product and write a review
    Client->>Security: POST /api/v1/products/{productId}/reviews + customer JWT
    Security->>Controller: Forward authenticated CUSTOMER principal
    Controller->>Service: createReview(productId, rating, comment, principal)
    Note over Service: The review author is resolved from the authentication principal, never from client input.
    alt The caller purchased and received the product and has no review for it yet
        Service->>Repo: Verify purchase and delivery history for (principal, product)
        Repo->>DB: SELECT order and shipment history used for eligibility
        DB-->>Repo: Delivered purchase rows
        Repo-->>Service: Eligible purchase confirmed
        Service->>Repo: Confirm no existing review for the unique (user, product) pair
        Repo->>DB: SELECT review by (user, product)
        DB-->>Repo: No existing review
        Service->>Repo: Persist the review with rating and comment
        Repo->>DB: INSERT review in one transaction
        DB-->>Repo: Persisted review
        Repo-->>Service: Review entity
        Service-->>Controller: Created result
        Controller-->>Client: 201 Created ApiResponse<ReviewDto>
        Client-->>Customer: Display the created review
    else Not purchased/received, duplicate review, or invalid rating
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Customer: Show the rejection
    end
    Note over Repo,DB: Order and shipment history is read for eligibility, and no direct order FK is modeled in the review table.
```
