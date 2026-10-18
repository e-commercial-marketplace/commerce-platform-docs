# AI Feature

## Purpose

Owns the Spring AI assistant, natural-language product-search mapping, read-only tool calling, chat
memory, RAG, and AI chat persistence inside the same Spring Boot monolith.

## MVP boundary

The first AI slice answers marketplace questions without requiring login: it maps a guest or
authenticated customer's natural-language request to a constrained public product search (via Tool
Calling on the catalog service) or to RAG-grounded policy/FAQ knowledge. Guest requests may be
stateless. Persisted sessions, order and shipment tools, voucher tools, and seller revenue analysis
are deferred.

The selected LLM provider is **DeepSeek**, configured on the backend via environment variables
(OpenAI-compatible API through Spring AI). Provider secrets never leave the backend. The AI feature
must never crash the app: when the provider is not configured or the provider call fails, the
backend returns `AI_UNAVAILABLE` (HTTP 503) and the client shows a "not available" state. The public
AI endpoint is rate-limited.

## Responsibilities

- Provide `POST /api/ai/ask` as a public endpoint (no login required for public data).
- Configure `ChatClient` with the DeepSeek provider from environment configuration.
- Return a controlled `AI_UNAVAILABLE` error when the provider is missing or failing.
- Persist `AiChatSession` and `AiMessage` history for authenticated customers.
- Expose public product-search tools first; add customer-owned order, shipment, tracking, and voucher
  tools later.
- Resolve the authenticated customer from Spring Security context for protected data; guests only
  receive public data and must never pass a user ID.
- Use RAG for policies/FAQ and tools for live transactional data.
- Add semantic product search/review summary later without enabling irreversible actions.
- Keep seller revenue analysis as planning only.

## Planned API

```text
POST /api/ai/ask                         # public for catalog search; customer scope when authenticated
GET  /api/ai/sessions                    # CUSTOMER only
GET  /api/ai/sessions/{id}/messages      # CUSTOMER and own session only
```

## Permissions

| Actor | Allowed operation | Constraint |
|---|---|---|
| Unauthenticated client | Ask about public products, FAQ, and policies | No login required. No order, shipment, voucher, or account data. Requests may be stateless and are rate-limited. |
| CUSTOMER | Search products; later ask about their own orders, shipments, and vouchers | Protected data is resolved from the authenticated principal. |
| SELLER/ADMIN | No AI access in the current scope | Seller revenue analysis is planning only. |
| AI model/client | Call read-only tools | Must not provide arbitrary `userId` access or direct repository bypass. |

## Focused class diagram

```mermaid
classDiagram
    class User {
        UUID id
        Role role
    }

    class AiChatSession {
        UUID id
        DateTime createdAt
    }

    class AiMessage {
        UUID id
        AiRole role
        String content
        Decimal confidence?
        DateTime createdAt
    }

    class AiRole {
        <<enumeration>>
        USER
        ASSISTANT
        SYSTEM
    }

    User "1" --> "0..*" AiChatSession : owns
    AiChatSession "1" *-- "0..*" AiMessage : turns
    AiMessage --> AiRole : role
```

## Tool and RAG boundary

```mermaid
flowchart LR
    AiService --> ChatClient
    ChatClient --> ProductTools[ProductTools]
    ChatClient --> OrderTools[OrderTools]
    ChatClient --> ShipmentTools[ShipmentTools]
    ChatClient --> VoucherTools[VoucherTools]
    ProductTools --> ExistingServices[Existing application services]
    OrderTools --> ExistingServices
    ShipmentTools --> ExistingServices
    VoucherTools --> ExistingServices
    ChatClient --> RAG[Policies, FAQ, and pgvector]
```

Use tools for current order status, shipment status, stock, voucher availability, and live price.
Use RAG for return policy, shipping policy, payment guide, voucher policy, seller rules, and FAQ.

Product-search flow:

```mermaid
flowchart LR
    UserText[User natural-language request] --> Extract[Extract ProductSearchQuery]
    Extract --> Validate[Validate allowed filters and limits]
    Validate --> Catalog[Catalog service]
    Catalog --> Results[Public product results]
    Results --> Answer[Grounded AI answer]
```

The AI must not invent product availability, price, or stock outside the catalog-service result.

## Forbidden initial actions

The initial AI feature must not expose `cancelOrder`, `refundPayment`, `changePrice`, `updateStock`,
`markDelivered`, or `deleteReview`.

## Data model

The persisted AI model stores customer conversations only. Guest product-search requests are public
and stateless, so they should not create a customer-owned session.

### Table relationships

- `users (1) -> ai_chat_sessions (many)`: an authenticated customer owns their saved AI sessions.
- `ai_chat_sessions (1) -> ai_messages (many)`: a session contains ordered user, assistant, or system turns.
- `ai_chat_sessions.userId -> users.id` is the ownership boundary; never accept the owner ID from the request body.

### Field meanings

| Table | Field | Meaning |
|---|---|---|
| `ai_chat_sessions` | `id` | Stable identifier for one persisted customer conversation. |
| `ai_chat_sessions` | `createdAt` | Time at which the session was created. |
| `ai_chat_sessions` | `userId` | Authenticated customer who owns the session. |
| `ai_messages` | `id` | Stable identifier for one conversation turn. |
| `ai_messages` | `role` | `USER`, `ASSISTANT`, or `SYSTEM`; controls how the turn is interpreted. |
| `ai_messages` | `content` | User input or the grounded assistant/system message. |
| `ai_messages` | `confidence` | Optional confidence metadata for the mapped answer; it is not an authorization decision. |
| `ai_messages` | `createdAt` | Time at which the turn was persisted. |
| `ai_messages` | `sessionId` | Parent `ai_chat_sessions` record. |

AI session/message fields are created by V13. V11 remains a historical placeholder; pgvector and AI
document/vector tables are deferred until the RAG schema is finalized.
