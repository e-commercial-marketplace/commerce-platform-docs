# Category - Create Category — Sequence Diagram

- **Use-case spec:** [create-category.vi.md](../../usecase-specification/admin/create-category.vi.md)
- **API route/status:** `POST /api/v1/admin/categories` — Predicted
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
    Admin->>Client: Start the create category use case
    Client->>Security: POST /api/v1/admin/categories
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: Execute create category
    alt Main flow succeeds
        Service->>Repo: Validate and create records for create category
        Repo->>DB: INSERT (transactional when required)
        DB-->>Repo: Domain row(s) or persistence result
        Repo-->>Service: Entity/result
        Note over Controller,Repo: Admin category write endpoints are Predicted in the Catalog API contract.
        Service-->>Controller: Use-case result
        Controller-->>Client: Success response defined by the contract (usually 201 Created)
        Client-->>Admin: Display the result
    else Validation, authorization, or business rule rejects the request
        Service-->>Controller: Propagate exception
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: Error ApiResponse per contract
        Client-->>Admin: Display the error
    end
```
