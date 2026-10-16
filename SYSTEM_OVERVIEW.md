# System Overview

## 1. Runtime architecture

```mermaid
flowchart LR
    Mobile[React Native Expo]
    Monolith[Spring Boot Modular Monolith]
    Database[(PostgreSQL)]
    Vector[(pgvector)]
    Vnpay[External VNPAY]
    Llm[External LLM Provider]

    Mobile -->|REST / WebSocket| Monolith
    Monolith --> Database
    Monolith --> Vector
    Monolith -->|Payment request and callback| Vnpay
    Monolith -->|Spring AI ChatClient| Llm

    subgraph Backend[Single JVM and deployment unit]
        Auth[auth]
        User[user]
        Catalog[catalog]
        Cart[cart]
        Order[order]
        Payment[payment]
        Shipment[shipment]
        Voucher[voucher]
        Review[review]
        Chat[chat]
        Ai[ai]
        Common[common]
    end

    Monolith --- Backend
    Auth -. normal Spring bean calls .-> User
    Order -. normal Spring bean calls .-> Cart
    Order -. normal Spring bean calls .-> Catalog
    Payment -. normal Spring bean calls .-> Order
    Shipment -. normal Spring bean calls .-> Order
    Ai -. read-only service calls .-> Catalog
    Ai -. read-only service calls .-> Order
    Ai -. read-only service calls .-> Shipment
    Ai -. read-only service calls .-> Voucher
```

## 2. Architectural decisions

- The backend is one Spring Boot application, one JVM process, one deployment unit, and one primary
  PostgreSQL database.
- Top-level organization is package-by-feature.
- Features communicate through normal Spring bean method calls inside the same JVM.
- Controllers remain thin; services/domain logic own business rules.
- Checkout, stock updates, voucher consumption, and payment callbacks use PostgreSQL transactions.
- Prepayment orders expire 24 hours after creation; checkout stock is restored exactly once. COD
  remains pending until all required non-cancelled seller shipments are delivered in the MVP.
- The first MVP uses `VND` and a fixed `0.00` shipping fee. Partial COD settlement for mixed
  `FAILED`/`RETURNED` seller outcomes is deferred.
- `FAILED` and `RETURNED` are seller-scoped shipment outcomes. `FAILED` does not restore stock;
  `RETURNED` restores that shipment's quantity after warehouse receipt.
- A shipment belongs to one order and one seller. MVP mapping uses whole order-item lines, with each
  order item assigned to at most one shipment; `shipment.orderId` must equal `orderItem.orderId`.
- Spring AI is an internal feature, not a separate service.
- The selected LLM provider is DeepSeek, called through Spring AI on the backend.
- AI starts with read-only tools and must not bypass normal authorization.
- React Native never calls the LLM provider directly. AI must degrade gracefully with
  `AI_UNAVAILABLE` when the provider is not configured or failing.

## 3. Client-to-backend flow

```mermaid
sequenceDiagram
    participant C as React Native Client
    participant API as Spring Boot API
    participant S as Feature Service
    participant DB as PostgreSQL
    participant EXT as External Provider

    C->>API: REST request (JWT when required)
    API->>API: Authenticate and authorize
    API->>S: Call feature service
    S->>DB: Read or transactional write
    DB-->>S: Domain data/result
    S-->>API: DTO or business exception
    API-->>C: Consistent API response
    API->>EXT: VNPAY or LLM call when required
    EXT-->>API: Callback or model result
```

## 4. AI flow

```mermaid
flowchart TD
    Guest[Guest] --> AiController[AiController]
    Customer[Authenticated CUSTOMER] --> AiController
    AiController --> AiService[AiService]
    AiService --> ChatClient[Spring AI ChatClient]
    ChatClient --> Query[Constrained ProductSearchQuery]
    Query --> Catalog[Catalog service]
    ChatClient --> Tools[Customer read-only tools]
    ChatClient --> Memory[Chat memory]
    ChatClient --> RAG[RAG retrieval]
    Tools --> ExistingServices[Existing application services]
    RAG --> Policies[Policies and FAQ]
    Tools --> LiveData[Customer-owned live data]
    Memory --> AiSession[AiChatSession and AiMessage for CUSTOMER]
    ChatClient --> LLM[External LLM provider]
```

RAG is for semi-static policy knowledge. Catalog/application services are the source for live data.
Guest requests are limited to public product search; seller and admin AI access is out of scope.

## 5. Feature boundaries

| Boundary | Owns | Must not bypass |
|---|---|---|
| `auth` | Registration, login, refresh, JWT, security | Spring Security and authenticated principal |
| `catalog` | Categories, products, stock, catalog browsing | Seller ownership and product activity rules |
| `cart` | Customer cart items and cart validation | Active product and stock validation |
| `order` | Checkout, order, order items, seller processing | Transaction, snapshot, and ownership rules |
| `payment` | COD and VNPAY attempts/callbacks | Signature validation and idempotency |
| `shipment` | Seller shipments and delivery state | Same-seller shipment rule and status invariants |
| `voucher` | Voucher validation and usage | Expiry, exhaustion, and concurrency rules |
| `review` | Eligibility and reviews | Purchase/delivery eligibility and uniqueness |
| `chat` | Human conversations and messages | Participant ownership and message consistency |
| `ai` | ChatClient, product-query mapping, tools, memory, RAG | Guest public search and CUSTOMER-only protected data |
| `common` | Shared response, exception, config, utility | Feature-specific business decisions |

## 6. External integrations

- PostgreSQL stores primary marketplace data.
- pgvector stores embeddings for the RAG phase.
- VNPAY handles online payment interaction.
- The selected DeepSeek provider (OpenAI-compatible API) supplies model responses through Spring AI.
- Provider secrets remain on the backend.
- Production error responses use safe `INTERNAL_ERROR` messages without framework binding details.
