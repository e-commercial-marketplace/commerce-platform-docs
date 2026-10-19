# Category - Update Category — Sequence Diagram

- **Use-case spec:** [update-category.vi.md](../../usecase-specification/admin/update-category.vi.md)
- **API route/status:** `PATCH /api/v1/admin/categories/{id}` — Predicted
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
    Admin->>Client: Start the update category use case
    Client->>Security: PATCH /api/v1/admin/categories/{id}
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute update category
    alt Main flow succeeds
        Service->>Repo: Load, validate ownership/state, and update for update category
        Repo->>DB: SELECT, validate, then UPDATE
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin category write endpoints are Predicted in the Catalog API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK response defined by the contract
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
