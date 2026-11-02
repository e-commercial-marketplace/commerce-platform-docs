# Category - Update Category — Sequence Diagram

- **Use-case spec:** [update-category.vi.md](../../usecase-specification/admin/update-category.vi.md)
- **API route/status:** `PATCH /api/v1/admin/categories/{id}` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AdminCategoryController
    participant Service as CategoryService
    participant Repo as CategoryRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Update Category
    Client->>Security: PATCH /api/v1/admin/categories/{id}
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: renameCategory(id, request)
        alt Request succeeds
            Service->>Repo: Find the category and check the name
            Repo->>DB: SELECT categories
            Service->>Repo: saveAndFlush the new name
            Repo->>DB: UPDATE categories
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<CategorySummaryResponse>
            Client-->>Admin: Display the result
        else Blank or duplicate name
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Admin: Display the error
        else Unknown category
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Admin: Display the error
        end
    end
```
