# 03. Architecture & Design Specification

## 1. Document Control

| Field | Value |
|---|---|
| Document ID | DOC-03 |
| Document title | Architecture & Design Specification |
| Project | E-Commerce Marketplace |
| Primary source | [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md) |
| Class model reference | [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml) |
| Status | Architecture and design baseline |
| Architecture style | Modular monolith |

## 2. Purpose

This document describes the target architecture, module boundaries, package organization, domain
model, key flows, security design, persistence design, external integrations, and test strategy.

The PlantUML class diagram is the reference for class attributes, enumerations, relationships, and
class-level constraints. This document summarizes that model and does not replace the source diagram.

## 3. Architecture Drivers

The design is driven by the following constraints from `PLAN.md`:

- the system remains one Spring Boot application, one JVM process, one deployment unit, and one
  primary PostgreSQL database;
- features are separated by business capability, not by one global technical-layer tree;
- cross-feature calls are normal Spring bean method calls inside the same JVM;
- controllers are thin and business rules are placed in services/domain logic;
- transactional consistency is required for checkout, stock, voucher consumption, and payment
  callbacks;
- prepayment orders expire 24 hours after creation and restore checkout stock exactly once; COD
  remains pending until all required seller shipments are delivered in the MVP;
- Spring AI remains inside the monolith and starts with read-only tools;
- React Native never calls an LLM provider directly.

## 4. System Context and Deployment View

```text
React Native (Expo)
        |
        | REST / WebSocket
        v
+------------------------------------------------+
|           Spring Boot Modular Monolith         |
|                                                |
| auth | user | catalog | cart | order           |
| payment | shipment | voucher | review          |
| chat | ai | common                             |
|                                                |
| AI module                                      |
|   ChatClient                                   |
|   + Tool Calling -> existing feature services  |
|   + Chat Memory                                |
|   + RAG -> pgvector                            |
+--------------------------+---------------------+
                           |
                           v
                 PostgreSQL + pgvector
                           |
                           +--> external VNPAY
                           +--> external LLM provider
```

### 4.1 Runtime components

| Component | Responsibility |
|---|---|
| React Native (Expo) | Presentation, navigation, client state, REST/WebSocket consumption |
| Spring Boot monolith | API, business rules, authorization, transactions, payment, shipment, chat, and AI orchestration |
| PostgreSQL | Primary marketplace persistence |
| Redis (optional) | Read cache for cart lines, the category list and seller shop pages; never the only copy of any data |
| pgvector | Embedding/vector storage for RAG |
| VNPAY | External online payment provider |
| LLM provider | External model provider accessed only by Spring AI backend |

### 4.2 Technology stack

| Layer | Technology | Status |
|---|---|---|
| Backend runtime | Spring Boot 3.4.4, Java 21, Maven 3.9+ | Current scaffold |
| Backend API | Spring Web MVC, Validation, Springdoc OpenAPI 2.7.0, Actuator | Current scaffold |
| Backend mapping | MapStruct 1.6.3 with Lombok binding for entity/DTO mapping | Current backend dependency |
| Backend persistence | Spring Data JPA/Hibernate, PostgreSQL 16, Flyway | Current scaffold |
| Backend cache | Spring Data Redis (Lettuce); Render Key Value in deployment | Optional; read cache (§5.4) |
| Backend security | Spring Security and JJWT 0.12.6 | Current dependency; flow is phased |
| Backend realtime | Spring WebSocket | Dependency present; human chat is phased |
| Mobile runtime | React Native with Expo | Scaffold; version TBD |
| Mobile HTTP | Axios | Planned frontend dependency |
| Mobile server state | TanStack Query | Planned frontend dependency |
| Mobile secure storage | Expo SecureStore | Planned frontend dependency |
| AI | Spring AI, external LLM provider, pgvector RAG | Planned backend phase |
| Testing | Spring Boot Test, JUnit 5, Testcontainers, H2 | Current backend scaffold |

The frontend has no `package.json` or installed dependencies yet. Its listed libraries are design
choices for the first Expo implementation, not evidence of a completed mobile build.

## 5. Logical Module Architecture

### 5.1 Feature modules

| Feature | Main responsibility | Planned internal layers |
|---|---|---|
| `auth` | Register, login, refresh, JWT, security | `controller`, `service`, `repository`, `dto`, `security`, `config` |
| `user` | User data, roles, seller status | `entity`, `repository`, `service`, `dto`, `mapper` |
| `catalog` | Categories, products, stock, customer catalog | `entity`, `repository`, `service`, `controller`, `dto`, `mapper` |
| `cart` | Customer cart items and cart validation | `entity`, `repository`, `service`, `controller`, `dto` |
| `order` | Orders, order items, checkout, seller order processing | `entity`, `repository`, `service`, `controller`, `dto`, `mapper` |
| `payment` | Payment records, COD, VNPAY integration | `entity`, `repository`, `service`, `controller`, `dto`, `vnpay` |
| `shipment` | Seller shipments, delivery method, tracking, status | `entity`, `repository`, `service`, `controller`, `dto` |
| `voucher` | Voucher data, validation, usage | `entity`, `repository`, `service`, `controller`, `dto` |
| `review` | Review eligibility and product reviews | `entity`, `repository`, `service`, `controller`, `dto` |
| `chat` | Human conversations, messages, WebSocket | `entity`, `repository`, `service`, `websocket`, `dto` |
| `ai` | ChatClient, tools, RAG, memory, AI persistence | `controller`, `service`, `repository`, `config`, `tools`, `rag`, `memory`, `entity`, `dto` |
| `common` | Shared errors, responses, config, utilities | `exception`, `response`, `config`, `util` |

### 5.2 Package structure

```text
com.marketplace
├── auth
│   ├── controller
│   ├── service
│   ├── repository
│   ├── dto
│   ├── security
│   └── config
├── user
│   ├── entity
│   ├── repository
│   ├── service
│   ├── dto
│   └── mapper
├── catalog
├── cart
├── order
├── payment
├── shipment
├── voucher
├── review
├── chat
├── ai
└── common
```

Each feature expands into the layers shown in section 5.1 when that responsibility is needed. The
architecture avoids a global `controller/`, `service/`, `repository/`, `entity/` layout.

### 5.3 Service and persistence call path

The planned synchronous request path is:

```text
React Native -> FeatureController -> FeatureService -> FeatureRepository -> PostgreSQL
```

`FeatureService` is the feature's service layer: the controller calls it directly in the same JVM, and
it owns use-case orchestration and business rules. It may call another feature through that feature's
service. Repositories own persistence access through Spring Data JPA. Provider calls (for example
VNPAY or the LLM provider) are feature-specific and appear only in the corresponding sequence
diagrams.

Entities hold data only: fields with their JPA mappings, Lombok accessors and a builder, and
`@PrePersist`/`@PreUpdate` callbacks that set `createdAt`/`updatedAt`. They contain no factory, state
transition or business-rule methods. A rule lives in the service of the feature that owns the data; another
feature that needs the rule calls that service.

This is the target architecture, not a claim about completed code: the current backend scaffold does
not yet contain the controller/service/repository layers.

### 5.4 Read cache (Redis)

#### Cart lines

PostgreSQL `cart_items` is the source of truth for carts. Redis only keeps a copy of each customer's cart
lines so `GET /cart` can skip that query; losing Redis never loses a cart.

- Key `cart:<userId>` is a Redis Hash: field = cart item id, value = product id, quantity and
  `createdAt`. Product name, price, stock and status are always read from PostgreSQL, so a seller's
  price change shows at once.
- Read (cache-aside): on a hit the lines come from Redis; on a miss they are read from PostgreSQL and
  written to Redis with a 30-minute time to live.
- Write: adding, updating or removing a cart item and checkout change PostgreSQL in their transaction as
  before, then delete `cart:<userId>` after the transaction commits. A rolled-back transaction leaves the
  cache untouched.
- Checkout always reads the selected cart lines from PostgreSQL, never from the cache.
- Accepted risk: a read that misses the cache while another request changes the cart can put the older
  lines back into Redis. The cart can then look stale for at most 30 minutes, until the next change or
  the time to live ends.

#### Catalog read results

Spring Cache stores these public read results in Redis as JSON:

| Cache | Endpoint | Key | Time to live | Cleared after commit by |
|---|---|---|---|---|
| `categories` | `GET /categories` | `all` | 1 hour | admin create, rename or delete of a category |
| `sellerShops` | `GET /sellers/{id}` | seller id | 10 minutes | that seller creating, updating or hiding a product; the seller changing their name; an admin updating, approving or suspending that seller |
| `products` | `GET /products/{id}` | product id | 30 minutes | any change of that product: seller update, hide or stock change; stock taken by checkout or given back (line cancelled, shipment returned, unpaid VNPAY order expired); a new review; any category rename (the detail shows the category name) |
| `productSearch` | `GET /products` | all search parameters | 60 seconds | never; it only expires |

- Cached values are written and cleared only after the transaction commits, so a rolled-back change
  never clears or fills the cache.
- "Not found" answers are not cached: a seller who is approved later appears at once.
- Every stock or rating change goes through `ProductService` (`decreaseStock`, `increaseStock`,
  `addRating`), which clears the product's detail entry, so no caller can forget it.
- Accepted risk: a search result page can be up to 60 seconds old, so a product hidden or repriced a
  moment ago can still be listed (its detail is already correct). The AI assistant and the shop page
  read products through the same search, so they share this delay. Cart, checkout and orders never
  read the cache.

#### Common rules

- All caches are off unless `APP_CACHE_ENABLED=true`. When Redis fails, the backend logs a warning and
  reads PostgreSQL; the API does not fail. Redis is left out of `/actuator/health`.
- Responses are the same with or without the cache.

## 6. Domain Model Design

### 6.1 Classes from the referenced diagram

The class model contains the following entities:

```text
User, Category, Product, CartItem, Order, OrderItem,
Payment, Voucher, Shipment, Review, Conversation, Message,
AiChatSession, AiMessage
```

The class model contains the following enumerations:

```text
Role, SellerStatus, OrderStatus, OrderItemStatus,
PaymentMethod, PaymentStatus, DeliveryMethod, ShipmentStatus,
AiRole
```

`AiRole` represents the role of an AI message: `USER`, `ASSISTANT`, or `SYSTEM`.

### 6.2 Core relationships

| Relationship | Design meaning |
|---|---|
| `User -> CartItem -> Product` | A customer owns cart items, each referring to one product. |
| `User -> Order -> OrderItem` | A customer places orders; an order owns one or more order items. |
| `OrderItem -> Product` | Each order item refers to the purchased product and stores product name/price snapshots. |
| `OrderItem -> User` | Each order item records its seller. |
| `User -> Product` | A seller owns the products they sell. |
| `Category -> Product` | A category contains products. |
| `Order -> Payment` | One order may have multiple payment attempts. |
| `Order -> Voucher` | An order may apply zero or one voucher. |
| `Order -> Shipment` | An order may produce multiple seller-specific shipments. |
| `Shipment -> OrderItem` | A shipment contains one or more order items from the same seller. |
| `Shipment -> User` | A shipment is owned/managed by one seller. |
| `User -> Review -> Product` | A customer writes reviews for products. |
| `Conversation -> Message` | A human conversation owns its messages. |
| `Conversation -> Product/Order` | A human conversation may refer to an optional product or order. |
| `User -> AiChatSession -> AiMessage` | AI chat persistence is separate from human chat. |

### 6.3 Domain constraints

The following constraints are design invariants:

- `sellerStatus` is required only when `role = SELLER`.
- A product has `price > 0` and `stock >= 0`.
- `active = false` means the product is soft-deleted or unlisted.
- Cart uniqueness is `(user, product)` and cart quantity is at least 1.
- Review uniqueness is `(user, product)` and rating is in `[1..5]`.
- A seller can modify only owned products, order items, and shipments.
- All order items in one shipment must belong to the same seller.
- `Shipment.seller == OrderItem.seller` for every item in that shipment.
- `Shipment.orderId == OrderItem.orderId` for every shipment mapping.
- An MVP order item is mapped to at most one shipment, using the complete order-item line; partial,
  split, and replacement shipments are deferred.
- `transactionId` is required and unique for VNPAY.
- A payment attempt can be retried by creating another payment row.
- `trackingCode` and `shippedAt` are required for `SHIPPED`.
- `deliveredAt` is required for `DELIVERED`.
- `FAILED` is a seller-scoped shipment outcome and never restores stock by itself.
- `RETURNED` means the package reached the seller warehouse; only then is that shipment's quantity
  restored.
- `OrderStatus` remains an order-level aggregate and does not collapse mixed seller outcomes.
- Direct order-item cancellation is allowed before shipment assignment. An assigned `PENDING` or
  `PREPARING` shipment uses an atomic cancellation path that marks mapped lines `CANCELLED` and
  releases unshipped stock exactly once; `SHIPPED` or later cannot be cancelled.
- Before payment completion, that transaction recalculates `Order.totalAmount` and pending
  `Payment.amount` from the remaining line snapshots. All-cancelled orders become `CANCELLED`; if
  delivered non-cancelled lines are all that remain, payment/order completion is reevaluated immediately.
- Money is represented in `VND`, and the first MVP uses a fixed `0.00` shipping fee.
- The MVP human chat rule is unique `(customer, seller)`.
- AI chat is separate from human chat and is read-only for Order, Payment, and Shipment operations.

## 7. State and Workflow Design

### 7.1 Customer checkout

```text
Cart
  -> validate active products
  -> validate stock
  -> validate voucher
  -> recalculate prices server-side
  -> create Order(PENDING)
  -> create OrderItems with name/price snapshots
  -> reserve or decrease stock
  -> set prepayment deadline to createdAt + 24 hours when the order requires prepayment
  -> save discount/shipping/total snapshots
  -> clear purchased cart items
```

The entire checkout operation is consistency-sensitive and should be transactional.

### 7.2 Seller fulfillment

```text
OrderItem PENDING
      -> CONFIRMED
      -> included in Shipment

PENDING/CONFIRMED
      -> CANCELLED
```

The service must compare the authenticated seller with `orderItem.seller.id` before changing state.

### 7.3 Shipment state

```text
PENDING
  -> PREPARING
  -> SHIPPED
      -> DELIVERED
      -> FAILED
          -> RETURNED

PENDING/PREPARING
  -> CANCELLED
```

`SHIPPED` requires `trackingCode` and `shippedAt`. `DELIVERED` requires `deliveredAt`. `FAILED` does
not restore stock. After the package reaches the seller warehouse, the shipment moves to `RETURNED`
and only that transition restores its shipped quantity. The order is set to `DELIVERED` only after
all required non-cancelled shipments are delivered; mixed seller outcomes remain visible per shipment.
Cancelling an assigned `PENDING`/`PREPARING` shipment releases unshipped stock and is not a
`RETURNED` operation.
It also recalculates the current payable amount without changing historical line snapshots.
For mixed terminal outcomes, the order remains an aggregate container; the API exposes the result per
shipment and does not introduce a new order-level failure enum.

### 7.4 Payment state

COD:

```text
Payment(method=COD, status=PENDING)
  -> COMPLETED when all required non-cancelled shipments are DELIVERED
```

The final required `DELIVERED` transition is the COD collection trigger handled by the shipment/payment
services in one transaction. The MVP records no partial COD settlement or refund for mixed
`FAILED`/`RETURNED` seller outcomes.

VNPAY:

```text
Create Payment(PENDING)
 -> redirect/create VNPAY request
 -> callback
      COMPLETED
      or FAILED
```

Callbacks must validate signatures/checksums and use a unique transaction/reference identifier for
idempotent processing.

## 8. API and Integration Design

### 8.1 REST boundaries

All planned REST groups are versioned with the same base prefix:

```text
/api/v1/auth
/api/v1/products
/api/v1/categories
/api/v1/seller/products
/api/v1/cart
/api/v1/orders
/api/v1/seller/order-items
/api/v1/vouchers
/api/v1/payments/vnpay
/api/v1/seller/shipments
/api/v1/seller/carriers
/api/v1/admin/carriers
/api/v1/products/{productId}/reviews
/api/v1/conversations
/api/v1/ai
```

Controllers expose transport concerns. Services enforce business rules, ownership, transaction
boundaries, and calls to other feature services.

The `/api/v1` prefix is added centrally for `@RestController`s of this application (`api.prefix` in
`application.yml`, applied by `WebConfig`), so controllers declare routes without it. Actuator,
OpenAPI/Swagger UI, and `/error` keep their unversioned paths, and the security matchers in
`SecurityConfig` use the prefixed URLs.

### 8.2 Human chat boundary

Human chat uses `WS /ws/chat` for real-time updates and REST endpoints for conversation/message
history. Message persistence and retry/idempotency remain inside the `chat` feature.

### 8.3 AI boundary

```text
AiController
    -> AiService
        -> ChatClient
            -> read-only tools
            -> RAG
            -> chat memory
```

AI tools call existing application services. They resolve the authenticated customer from Spring
Security context for protected data and do not accept arbitrary protected `userId` values from the
model or client. Guest AI access is limited to public product search.

## 9. AI Design

### 9.1 Delivery sequence

```text
ChatClient
 -> read-only tool calling
 -> AiChatSession/AiMessage memory
 -> RAG + pgvector
 -> semantic product search/review summary
```

### 9.2 Read-only tools

| Tool group | Planned capabilities |
|---|---|
| `ProductTools` | `searchProducts(ProductSearchQuery)`, `getProduct` for public catalog data |
| `OrderTools` | Later customer-only `getMyOrders`, `getMyOrder(orderId)` |
| `ShipmentTools` | Later customer-only `getShipmentStatus(orderId)`, `getTrackingInformation(orderId)` |
| `VoucherTools` | Later customer-only `checkVoucher(code)` |

The product-search boundary is:

```text
Natural-language request
  -> constrained ProductSearchQuery
  -> validation of fields, sort, and page limits
  -> catalog service search
  -> grounded natural-language answer
```

Seller revenue analysis is planning only.

The initial AI layer must not expose `cancelOrder`, `refundPayment`, `changePrice`, `updateStock`,
`markDelivered`, or `deleteReview`.

### 9.3 RAG boundary

RAG is for semi-static content such as return policy, shipping policy, payment guide, voucher policy,
seller rules, and FAQs.

Live order status, shipment status, stock, voucher availability, and product price use tool calling
instead of RAG.

## 10. Security Design

- Use Spring Security with JWT access and refresh flow.
- Hash passwords with `PasswordEncoder`; never store plaintext.
- Enforce role checks through endpoint/service authorization.
- Enforce ownership checks in addition to roles.
- Prevent a seller from managing another seller's product, order item, or shipment.
- Limit customer order/shipment queries to the authenticated customer.
- Validate all VNPAY callback signatures/checksums.
- Make payment callbacks idempotent.
- Resolve protected AI tool identity from Spring Security context; permit guests only on public
  product-search operations.
- Never trust role or user IDs supplied by React Native.
- Keep provider API keys on the backend.
- In production, disable framework error-message and binding-error inclusion; return safe
  `INTERNAL_ERROR` responses for unexpected failures.

## 11. Persistence and Migration Design

Use Flyway for ordered schema changes. The planned migration order is:

```text
V1  users
V2-V11 historical placeholders (do not edit after Flyway records them)
V12 refresh_tokens
V13 domain tables: categories, products, cart_items, orders/items, vouchers, payments,
    shipments/mapping, reviews, conversations/messages, ai_chat_sessions/messages
V14 password_reset_codes (Phase 1 addition)
V15 cart_items.created_at (Phase 3 cart ordering)
V16+ pgvector extension and AI document/vector tables (deferred until the RAG schema is finalized)
```

Important constraints and indexes include:

- unique `user.email`;
- unique `category.name`;
- unique `(cart_item.user_id, cart_item.product_id)`;
- unique VNPAY transaction/reference where applicable;
- unique `(review.user_id, review.product_id)`;
- indexes on `product.seller_id`, `order.status`, `order_item.seller_id`, `shipment.seller_id`, and
  `shipment.status`.
- index on `order.created_at` for the 24-hour prepayment-order expiration scan.

Money values use `BigDecimal`.

## 12. Frontend Design Boundary

The React Native client is organized by technical responsibility under `frontend/src`. A file name
identifies its feature (for example `useCart`, `cartService`, `CartScreen`), mirroring the backend's
package-by-layer convention.

```text
src
├── api            Axios client, query client, query keys, error normalization
├── services       one function per endpoint; tokenStorage (SecureStore)
├── hooks          TanStack Query hooks (useCart, useOrders, useCheckout, ...)
├── screens        one screen per file
├── components     shared UI; components/seller for seller fulfillment
├── navigation     AppNavigator (root stack), MainTabs, route param types
├── context        AuthContext (client session state)
├── utils          pure helpers and DTO -> UI model mappers
├── types          API DTOs and UI models
└── constants      theme and shared configuration
```

### 12.1 Layers and request flow

```text
Screen -> Hook (TanStack Query) -> Service -> apiClient (Axios) -> Spring Boot /api/v1
```

| Backend layer | Frontend layer | Responsibility |
|---|---|---|
| Controller | Screen | Renders UI and handles user interaction; holds no business rules. |
| Security filter | `apiClient` | Attaches the Bearer token, rotates the refresh token once on `401`, normalizes errors. |
| Service | Service (`services/`) | One function per endpoint: path, payload, envelope unwrapping. |
| Repository | Hook (`hooks/`) | Server-state cache, loading, refetch, invalidation. |
| Mapper | `utils/*Mapper.ts` | Converts backend DTOs to UI models (for example decimal strings to numbers). |
| Database | Spring Boot API | The backend is the source of truth; the client has no database of its own. |

### 12.2 Implementation rules

1. Screens never call Axios directly; they use hooks.
2. Services contain endpoint paths and payload mapping only, and always use the shared `apiClient` and `unwrapData`. The one exception is the Cloudinary file upload in `uploadService`, which uses plain `axios` so the Bearer token never leaves the backend's origin (Expo's global `fetch` does not accept React Native's `{ uri, name, type }` file parts).
3. Hooks own server state. Server data is not copied into a React Context.
4. After a mutation the hook stores the authoritative response or invalidates the related query. There are no optimistic updates.
5. Query keys are built only in `api/queryKeys.ts`.
6. Mutations are not retried automatically. Sensitive actions such as checkout also guard against a double tap in the same tick.
7. `AuthContext` holds only the client session. The cart cache is purged whenever the identity changes.
8. The UI may hide actions (for example the cart is enabled only for `CUSTOMER`), but Spring Boot enforces authorization and ownership.
9. A new route is declared in `navigation/types.ts` together with its `AppNavigator` entry.
10. Tests live next to the code in `__tests__` folders.

React Native owns presentation and client state. Spring Boot remains the authority for authorization,
even when the mobile UI hides an action.

## 13. Test Design

### 13.1 Backend Test Design (Planned)

The backend architecture must eventually be verified with:

- unit tests;
- integration tests;
- security ownership tests;
- payment idempotency tests;
- multi-seller order/shipment tests;
- AI tool authorization tests;
- product search and pagination performance checks.

The most important cross-module tests cover seller isolation, order price snapshots, payment retry,
duplicate callback handling, voucher concurrency, review eligibility, and AI data isolation.

All items in this subsection are **Planned**. They describe design intent and are not implemented
yet. No backend integration, idempotency, ownership, or performance test exists in the current
milestone.

### 13.2 Frontend Testing (Implemented)

The React Native / Expo client currently has **45 automated frontend tests** across 8 suites,
classified as:

| Category | Count |
|---|---|
| Unit Test | 18 |
| Component Test | 13 |
| Hook/Context Test | 12 |
| Screen/Rendering Test | 2 |
| **Total** | **45** |

The stack is Jest `~29.7.0` with the `jest-expo` `~57.0.6` preset and
`@testing-library/react-native` `12.8.1`.

Verified result for frontend commit `36e02a6` (merged via `866897b`): 8 suites passed, 45 tests
passed, 0 failed, 0 skipped, 0 snapshots.

These are unit, component, hook/context, and limited screen-rendering tests only. They do not verify
real backend integration, server-side cart behavior, checkout, orders, payment, shipment, reviews,
voucher, chat, AI, or notifications.

Full test case specification, coverage, and traceability:
[`04-testing/README.md`](../04-testing/README.md).

## 14. Architecture Decisions and Exclusions

The design intentionally excludes the following until a later requirement justifies them:

- microservices;
- API Gateway;
- service discovery;
- Feign or inter-service REST;
- Kafka;
- Saga;
- distributed transactions;
- initial AI write actions.

These exclusions preserve a single deployable monolith and keep business operations inside normal
Spring transaction and authorization boundaries.
