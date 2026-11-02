# AI - View AI Chat History — Sequence Diagram

- **Use-case spec:** [view-ai-chat-history.vi.md](../../usecase-specification/customer/view-ai-chat-history.vi.md)
- **API route/status:** `GET /api/v1/ai/sessions, GET /api/v1/ai/sessions/{id}/messages` — Confirmed
- **Feature API contract:** [features/ai/API_CONTRACT.md](../../features/ai/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as AiController
    participant Service as AiService
    participant Sessions as AiChatSessionRepository
    participant Messages as AiMessageRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: View AI Chat History
    Client->>Security: GET /api/v1/ai/sessions, GET /api/v1/ai/sessions/{id}/messages
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: listSessions, then listSessionMessages
        alt Request succeeds
            Service->>Sessions: List the caller's sessions
            Sessions->>DB: SELECT ai_chat_sessions
            Service->>Sessions: Find the chosen session of this caller
            Service->>Messages: findBySessionId, oldest first
            Messages->>DB: SELECT ai_messages
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<List<AiChatSessionResponse>> or ApiResponse<List<AiMessageResponse>>
            Client-->>Customer: Display the result
        else Unknown or foreign session
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Customer: Display the error
        end
    end
```
