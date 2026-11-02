# Category - Manage Categories — Sequence Diagram

- **Use-case spec:** [manage-categories.vi.md](../../usecase-specification/admin/manage-categories.vi.md)
- **API route/status:** `GET /api/v1/categories, POST/PATCH/DELETE /api/v1/admin/categories` — Confirmed
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
    Admin->>Client: Manage Categories
    Client->>Security: GET /api/v1/categories, POST/PATCH/DELETE /api/v1/admin/categories
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Admin: Display the error
    else Caller is not ADMIN
        Security-->>Client: 403 FORBIDDEN
        Client-->>Admin: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated ADMIN principal
        Controller->>Service: Run the chosen category operation
        Note over Client,Service: See view-categories, create-category, update-category and delete-category
        Service->>Repo: Read or change categories
        Repo->>DB: SELECT, INSERT, UPDATE or DELETE categories
        Service-->>Controller: Use-case result
        Controller-->>Client: Response of the chosen operation
        Client-->>Admin: Display the result
    end
```
