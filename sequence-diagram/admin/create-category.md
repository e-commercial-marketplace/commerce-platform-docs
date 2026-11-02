# Category - Create Category — Sequence Diagram

- **Use-case spec:** [create-category.vi.md](../../usecase-specification/admin/create-category.vi.md)
- **API route/status:** `POST /api/v1/admin/categories` — Confirmed
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
    Admin->>Client: Create Category
    Client->>Security: POST /api/v1/admin/categories
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: createCategory(request)
        alt Request succeeds
            Service->>Repo: existsByNameIgnoreCase
            Service->>Repo: saveAndFlush the category
            Repo->>DB: INSERT categories
            Service-->>Controller: Use-case result
            Controller-->>Client: 201 Created ApiResponse<CategorySummaryResponse>
            Client-->>Admin: Display the result
        else Blank or duplicate name
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Admin: Display the error
        end
    end
```
