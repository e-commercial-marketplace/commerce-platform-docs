# Category - View Categories — Sequence Diagram

- **Use-case spec:** [view-categories.vi.md](../../usecase-specification/admin/view-categories.vi.md)
- **API route/status:** `GET /api/v1/categories` — Confirmed
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as CategoryController
    participant Service as CategoryService
    participant Cache as Redis cache
    participant Repo as CategoryRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: View Categories
    Client->>Security: GET /api/v1/categories
    Security->>Controller: Forward the request
    Controller->>Service: listCategories()
    Service->>Cache: Read the categories entry
    Service->>Repo: On a miss, findAllByOrderByNameAsc
    Repo->>DB: SELECT categories
    Service-->>Controller: Use-case result
    Controller-->>Client: 200 OK ApiResponse<List<CategorySummaryResponse>>
    Client-->>Admin: Display the result
```
