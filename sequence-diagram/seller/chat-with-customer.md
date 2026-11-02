# Chat - Chat with Customer — Sequence Diagram

- **Use-case spec:** [chat-with-customer.vi.md](../../usecase-specification/seller/chat-with-customer.vi.md)
- **API route/status:** `GET/POST /api/v1/conversations, GET/POST /api/v1/conversations/{id}/messages, PATCH /api/v1/conversations/{id}/read, WebSocket /ws/chat` — Confirmed
- **Feature API contract:** [features/chat/API_CONTRACT.md](../../features/chat/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as ChatController
    participant Service as ChatService
    participant Conversations as ConversationRepository
    participant Messages as MessageRepository
    participant Push as WebSocket /ws/chat
    participant Other as Customer
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Chat with Customer
    Client->>Security: GET/POST /api/v1/conversations, GET/POST /api/v1/conversations/{id}/messages, PATCH /api/v1/conversations/{id}/read, WebSocket /ws/chat
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Caller is not SELLER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Seller: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: openConversation, listMessages, markRead, sendMessage
        alt Request succeeds
            Service->>Conversations: Find by customer and seller, or create (customerId from the body)
            Conversations->>DB: SELECT or INSERT conversations
            Service->>Messages: findByConversationId, newest first
            Messages->>DB: SELECT messages
            Service->>Messages: markReadByRecipient
            Messages->>DB: UPDATE messages SET read
            Note over Client,Service: Sending is POST /conversations/{id}/messages with an Idempotency-Key header
            Service->>Messages: Save, or return the message stored under this key
            Messages->>DB: INSERT messages, UPDATE last_message_at
            Service-)Push: After commit, push the message to both sides
            Push-)Other: message event
            Service-->>Controller: Use-case result
            Controller-->>Client: 201 Created ApiResponse<MessageResponse>
            Client-->>Seller: Display the result
        else Empty message or missing Idempotency-Key
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Seller: Display the error
        else Unknown participant, or caller not in this conversation
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Seller: Display the error
        end
    end
```
