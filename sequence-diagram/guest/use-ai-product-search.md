# AI - Search Products with AI Assistant — Sequence Diagram

- **Use-case spec:** [use-ai-product-search.vi.md](../../usecase-specification/guest/use-ai-product-search.vi.md)
- **API route/status:** `POST /api/v1/ai/ask` — Planned
- **Feature API contract:** [features/ai/API_CONTRACT.md](../../features/ai/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AiController
    participant Service as AiService
    participant ChatClient as Spring AI ChatClient
    participant LLM as External DeepSeek provider
    participant CatalogService as CatalogService
    participant ProductRepo as ProductRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Guest->>Client: Ask a question about public products
    Client->>Security: POST /api/v1/ai/ask (public guest request)
    Security->>Controller: Forward public/authenticated request
    Controller->>Service: ask(message, optional sessionId)
    Service->>ChatClient: Convert the question to a constrained product query
    ChatClient->>LLM: Extract structured search criteria
    LLM-->>ChatClient: ProductSearchQuery
    ChatClient-->>Service: Validated search criteria
    Service->>CatalogService: Search public catalog with constrained criteria
    CatalogService->>ProductRepo: Find active, publicly visible products
    ProductRepo->>DB: SELECT matching catalog rows
    DB-->>ProductRepo: Product results
    ProductRepo-->>CatalogService: Product DTO source data
    CatalogService-->>Service: Grounding data
    Service->>ChatClient: Generate an answer grounded only in returned products
    ChatClient->>LLM: Compose grounded natural-language answer
    LLM-->>ChatClient: Answer text
    ChatClient-->>Service: Grounded answer
    Note over Service,DB: Guest search is stateless, it must not create a customer-owned AI session.
    Service-->>Controller: Result DTO
    Controller-->>Client: 200 OK ApiResponse<AiProductSearchResponse>
    Client-->>Guest: Display grounded results and answer
    opt Provider/model failure
        Service-->>Controller: Exception
        Controller->>Handler: Normalize error without leaking secrets
        Handler-->>Client: Safe error response per AI contract
        Client-->>Guest: Show unavailable/error notice
    end
```
