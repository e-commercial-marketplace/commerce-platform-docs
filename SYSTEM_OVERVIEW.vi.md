# Tổng quan hệ thống

## 1. Kiến trúc runtime

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

## 2. Quyết định kiến trúc

- Backend là một ứng dụng Spring Boot, một JVM process, một deployment unit và một primary
  PostgreSQL database.
- Tổ chức top-level là package-by-feature.
- Các feature giao tiếp qua normal Spring bean method calls trong cùng JVM.
- Controllers giữ thin; services/domain logic sở hữu business rules.
- Checkout, stock updates, voucher consumption và payment callbacks dùng PostgreSQL transactions.
- Prepayment orders hết hạn 24 giờ sau khi tạo; checkout stock được khôi phục đúng một lần. COD
  vẫn pending cho đến khi mọi seller shipments required còn non-cancelled đều delivered trong MVP.
- MVP đầu tiên dùng `VND` và fixed `0.00` shipping fee. Partial COD settlement cho các
  `FAILED`/`RETURNED` seller outcomes hỗn hợp được hoãn lại.
- `FAILED` và `RETURNED` là seller-scoped shipment outcomes. `FAILED` không khôi phục stock;
  `RETURNED` khôi phục quantity của shipment đó sau khi warehouse receipt.
- Một shipment thuộc về một order và một seller. Ánh xạ MVP dùng whole order-item lines, với mỗi
  order item được gán cho tối đa một shipment; `shipment.orderId` phải bằng `orderItem.orderId`.
- Spring AI là internal feature, không phải service riêng biệt.
- AI bắt đầu với read-only tools và không được bypass authorization thông thường.
- React Native không bao giờ gọi OpenAI hoặc Gemini trực tiếp.

## 3. Luồng client-backend

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

## 4. Luồng AI

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

RAG dùng cho semi-static policy knowledge. Catalog/application services là source cho live data.
Guest requests bị giới hạn ở public product search; seller và admin AI access ngoài scope.

## 5. Ranh giới feature

| Boundary | Owns | Must not bypass |
|---|---|---|
| `auth` | Registration, login, refresh, JWT, security | Spring Security và authenticated principal |
| `catalog` | Categories, products, stock, catalog browsing | Seller ownership và product activity rules |
| `cart` | Customer cart items và cart validation | Active product và stock validation |
| `order` | Checkout, order, order items, seller processing | Transaction, snapshot và ownership rules |
| `payment` | COD và VNPAY attempts/callbacks | Signature validation và idempotency |
| `shipment` | Seller shipments và delivery state | Same-seller shipment rule và status invariants |
| `voucher` | Voucher validation và usage | Expiry, exhaustion và concurrency rules |
| `review` | Eligibility và reviews | Purchase/delivery eligibility và uniqueness |
| `chat` | Human conversations và messages | Participant ownership và message consistency |
| `ai` | ChatClient, product-query mapping, tools, memory, RAG | Guest public search và CUSTOMER-only protected data |
| `common` | Shared response, exception, config, utility | Feature-specific business decisions |

## 6. Tích hợp bên ngoài

- PostgreSQL lưu primary marketplace data.
- pgvector lưu embeddings cho RAG phase.
- VNPAY xử lý online payment interaction.
- OpenAI/Gemini-compatible provider được chọn cung cấp model responses qua Spring AI.
- Provider secrets giữ trên backend.
- Production error responses dùng safe `INTERNAL_ERROR` messages không kèm framework binding details.
