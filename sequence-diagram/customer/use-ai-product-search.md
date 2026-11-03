# AI - Search Products with AI Assistant — Sequence Diagram

- **Use-case spec:** [use-ai-product-search.vi.md](../../usecase-specification/customer/use-ai-product-search.vi.md)
- **API route/status:** `POST /api/v1/ai/ask` — Confirmed
- **Feature API contract:** [features/ai/API_CONTRACT.md](../../features/ai/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AiController
    participant Service as AiService
    participant Limiter as AiRateLimiter
    participant SessionRepo as AiChatSessionRepository
    participant ChatClient as Spring AI ChatClient
    participant Memory as AiChatMemory
    participant LLM as OpenAI-compatible provider
    participant Tool as ProductSearchTools
    participant Rules as RuleBasedAnswerer
    participant Catalog as ProductService
    participant DB as PostgreSQL
    Customer->>Client: Ask a question about products
    Client->>Security: POST /api/v1/ai/ask (Bearer token, optional sessionId)
    Security->>Controller: Authenticated CUSTOMER
    Controller->>Service: answerQuestion(customer, message, sessionId)
    Service->>Limiter: Count this question for user:<id>
    alt More than 60 questions this minute
        Limiter-->>Service: BLOCKED
        Service-->>Client: 429 RATE_LIMITED
    end
    Limiter-->>Service: AI (1-20) or RULES (21-60)
    alt sessionId is null
        Service->>SessionRepo: Create a session owned by the customer
    else sessionId given
        Service->>SessionRepo: findByIdAndUserId(sessionId, customer)
        SessionRepo-->>Service: Not found or another customer's session
        Service-->>Client: 404 RESOURCE_NOT_FOUND
    end
    alt Level AI and provider configured
        Service->>ChatClient: Prompt with system rules, tool and memory advisor (conversation = session id)
        ChatClient->>Memory: Last 10 messages of the session
        Memory->>DB: SELECT ai_messages
        ChatClient->>LLM: System prompt, history, question, tool definition
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
        ChatClient->>Memory: Append user and assistant messages
        Memory->>DB: INSERT ai_messages
        ChatClient-->>Service: Answer, answeredBy = AI
    else Level RULES, provider missing, cooling down after 429, or provider call failed
        Note over Service: APP_AI_RULES_ENABLED=false: 429 at level RULES, otherwise 503 AI_UNAVAILABLE instead of this branch
        Service->>Rules: answer(message)
        Rules->>Catalog: search with filters parsed from the message
        Catalog->>DB: SELECT products
        Rules-->>Service: Templated, canned or hint answer
        Service->>Memory: Append user and assistant messages
        Memory->>DB: INSERT ai_messages
    end
    Service-->>Controller: AiProductSearchResponse (answer, query, products, sessionId, answeredBy)
    Controller-->>Client: 200 OK
    Client-->>Customer: Show the answer and product cards
```
