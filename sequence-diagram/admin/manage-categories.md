# Category - Manage Categories — Sequence Diagram

- **Use-case spec:** [manage-categories.vi.md](../../usecase-specification/admin/manage-categories.vi.md)
- **API route/status:** `GET /api/v1/categories; POST/PATCH/DELETE /api/v1/admin/categories` — Mixed: confirmed public read + predicted admin writes
- **Feature API contract:** [features/catalog/API_CONTRACT.md](../../features/catalog/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

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
    Admin->>Client: Start the manage categories use case
    Client->>Security: GET /api/v1/categories, POST/PATCH/DELETE /api/v1/admin/categories
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute manage categories
    alt Main flow succeeds
        Service->>Repo: Query or persist data required to manage categories
        Repo->>DB: Read or write as the selected operation requires (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: The list path is public, admin create/update/delete paths are Predicted.
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
