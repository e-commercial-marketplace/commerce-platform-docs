# Chat - Chat with Seller — Sequence Diagram

- **Use-case spec:** [chat-with-seller.vi.md](../../usecase-specification/customer/chat-with-seller.vi.md)
- **API route/status:** `WebSocket /ws/chat (message frames are phased)` — Phased
- **Feature API contract:** [features/chat/API_CONTRACT.md](../../features/chat/API_CONTRACT.md)

> **Target sequence:** endpoint status is not fully Confirmed (see API contract link). Layer names are planned placeholders; the backend scaffold does not yet implement them.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as WebSocket JWT handshake
    participant Endpoint as ChatWebSocketEndpoint
    participant Service as ChatService
    participant ConversationRepo as ConversationRepository
    participant MessageRepo as MessageRepository
    participant DB as PostgreSQL
    Customer->>Client: Open conversation and send a message
    Client->>Security: Connect to /ws/chat with JWT handshake
    Security->>Endpoint: Forward authenticated CUSTOMER/SELLER principal
    Endpoint->>Service: Send message for the participant-owned conversation
    alt The sender is a participant of the conversation
        Service->>ConversationRepo: Verify conversation participation
        ConversationRepo->>DB: SELECT conversation by id and principal
        DB-->>ConversationRepo: Owned conversation or none
        Service->>MessageRepo: Persist message and update conversation timestamp
        MessageRepo->>DB: INSERT message / UPDATE conversation
        DB-->>MessageRepo: Persisted message
        MessageRepo-->>Service: Message result
        Service-->>Endpoint: WebSocket event
        Endpoint-->>Client: Deliver message event to authorized participant
        Client-->>Customer: Display the message
    else Handshake is invalid, the conversation is unknown, or the sender is not a participant
        Service-->>Endpoint: Error event without persisting a message
        Endpoint-->>Client: Error frame (not authorized or conversation not found)
        Client-->>Customer: Show the chat error
    end
    Note over Client,Endpoint: WebSocket setup, subscription paths, and exact frames remain phased in the chat contract.
```
