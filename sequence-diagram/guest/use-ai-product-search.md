# AI - Search Products with AI Assistant — Sequence Diagram

- **Use-case spec:** [use-ai-product-search.vi.md](../../usecase-specification/guest/use-ai-product-search.vi.md)
- **API route/status:** `POST /api/v1/ai/ask` — Confirmed
- **Feature API contract:** [features/ai/API_CONTRACT.md](../../features/ai/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Guest
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AiController
    participant Service as AiService
    participant Limiter as AiRateLimiter
    participant ChatClient as Spring AI ChatClient
    participant LLM as OpenAI-compatible provider
    participant Tool as ProductSearchTools
    participant Rules as RuleBasedAnswerer
    participant Catalog as ProductService
    participant DB as PostgreSQL
    Guest->>Client: Ask a question about products
    Client->>Security: POST /api/v1/ai/ask (no token, sessionId = null)
    Security->>Controller: Public request
    Controller->>Service: answerQuestion(guest, message, null)
    alt Guest sent a sessionId
        Service-->>Client: 401 AUTHENTICATION_REQUIRED
    end
    Service->>Limiter: Count this question for ip:<client IP>
    alt More than 60 questions this minute
        Limiter-->>Service: BLOCKED
        Service-->>Client: 429 RATE_LIMITED
    end
    Limiter-->>Service: AI (1-20) or RULES (21-60)
    alt Level AI and provider configured
        Service->>ChatClient: Prompt with system rules and the tool, no memory
        ChatClient->>LLM: System prompt, question, tool definition
        opt Broad request
            LLM-->>ChatClient: Call listCategories()
            ChatClient->>Tool: listCategories()
            Tool-->>ChatClient: Category names
        end
        LLM-->>ChatClient: Call searchProducts(text, category, prices, rating, sort)
        ChatClient->>Tool: searchProducts(...)
        Tool->>Catalog: search (top 5 listed products)
        Catalog->>DB: SELECT products
        Tool-->>ChatClient: Product results (also kept for the response)
        ChatClient->>LLM: Tool results
        LLM-->>ChatClient: Answer grounded in the results
        ChatClient-->>Service: Answer, answeredBy = AI
    else Level RULES, provider missing, cooling down after 429, or provider call failed
        Service->>Rules: answer(message)
        Rules->>Catalog: search with filters parsed from the message
        Catalog->>DB: SELECT products
        Rules-->>Service: Templated, canned or hint answer
    end
    Note over Service: Nothing is stored for guests
    Service-->>Controller: AiProductSearchResponse (sessionId = null)
    Controller-->>Client: 200 OK
    Client-->>Guest: Show the answer and product cards
```
