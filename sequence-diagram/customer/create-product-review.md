# Review - Create Product Review — Sequence Diagram

- **Use-case spec:** [create-product-review.vi.md](../../usecase-specification/customer/create-product-review.vi.md)
- **API route/status:** `POST /api/v1/products/{productId}/reviews` — Confirmed
- **Feature API contract:** [features/review/API_CONTRACT.md](../../features/review/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ReviewController
    participant Service as ReviewService
    participant Items as OrderItemRepository
    participant Repo as ReviewRepository
    participant Products as ProductService
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Create Product Review
    Client->>Security: POST /api/v1/products/{productId}/reviews
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: createReview(callerId, productId, request)
        alt Request succeeds
            Service->>Items: hasCompletedPurchase(customer, product)
            Items->>DB: SELECT order_items of COMPLETED orders
            Service->>Repo: saveAndFlush the review
            Repo->>DB: INSERT reviews
            Service->>Products: addRating (clears the product detail cache)
            Products->>DB: UPDATE products in one statement
            Service-->>Controller: Use-case result
            Controller-->>Client: 201 Created ApiResponse<ReviewResponse>
            Client-->>Customer: Display the result
        else Invalid rating or comment
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Customer: Display the error
        else Unknown or hidden product
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Customer: Display the error
        else No completed purchase
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 REVIEW_NOT_ELIGIBLE
            Client-->>Customer: Display the error
        else Already reviewed
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 REVIEW_ALREADY_EXISTS
            Client-->>Customer: Display the error
        end
    end
```
