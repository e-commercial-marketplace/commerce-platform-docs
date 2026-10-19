# Category - View Categories — Sequence Diagram

- **Use-case spec:** [view-categories.vi.md](../../usecase-specification/admin/view-categories.vi.md)
- **API route/status:** `GET /api/v1/categories (public route; admin-specific listing is not specified)` — Confirmed public read
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as CategoryController
    participant Service as CatalogService
    participant Repo as CategoryRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Admin->>Client: Start the view categories use case
    Client->>Security: GET /api/v1/categories (public route, admin-specific listing is not specified)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute view categories
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to view categories
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: GET /api/v1/categories is a confirmed public route, an Admin-only category list is not separately specified.
        Service-->>Controller: Use-case result
        Controller-->>Client: Contract-defined success response for the selected endpoint
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
