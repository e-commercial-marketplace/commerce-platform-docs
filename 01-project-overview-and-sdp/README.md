# 01. Project Overview & Software Development Plan

## 1. Document Control

| Field | Value |
|---|---|
| Document ID | DOC-01 |
| Document title | Project Overview & Software Development Plan |
| Project | E-Commerce Marketplace |
| Source baseline | [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md) |
| Supporting reference | [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml) |
| Status | Planning baseline |
| Architecture baseline | Modular monolith |

## 2. Purpose

This document defines the project context, product scope, technical direction, development sequence,
quality gates, and completion criteria for the E-Commerce Marketplace.

The document is limited to the planning information in `PLAN.md`. It does not claim that a planned
feature is implemented.

## 3. Project Overview

The product is an e-commerce marketplace implemented as a modular monolith:

- one Spring Boot application;
- one JVM process;
- one deployment unit;
- one primary PostgreSQL database.

The mobile client is a React Native application built with Expo. The backend owns authorization,
business rules, transactions, payment callbacks, shipment rules, human chat, and AI integration.

The marketplace supports multi-seller orders. One order may contain order items from multiple sellers.
Shipments are created per seller and may group multiple order items from that seller.

## 4. Product Scope

### 4.1 Customer capabilities

Customers can:

- browse, search, filter, and paginate products;
- manage cart items;
- place orders;
- pay using COD or VNPAY;
- track shipments;
- review products after the purchase and delivery conditions are satisfied;
- chat with sellers;
- use the marketplace AI assistant.

### 4.2 Seller capabilities

Sellers can:

- manage their own products and stock;
- process their own order items;
- choose platform delivery or self delivery;
- create and manage their own shipments;
- chat with customers.

### 4.3 Admin capabilities

Admins can:

- manage users, categories, and vouchers;
- moderate violations;
- monitor marketplace operations;
- configure platform delivery;
- perform administrative actions such as refund approval.

## 5. Development Constraints

The following decisions are mandatory for the planned system:

1. Keep the system as a modular monolith. Do not split it into microservices.
2. Do not introduce API Gateway, service discovery, Feign, internal REST, Kafka, Saga, or distributed
   transaction patterns unless a later requirement explicitly needs them.
3. Organize the backend by technical layer (`controller`, `service`, `repository`, `entity`, `dto`, ...); a class name identifies its feature.
4. Keep controllers thin. Business rules belong in services or domain logic.
5. Use PostgreSQL transactions for checkout, stock updates, voucher consumption, and payment callbacks.
6. Keep Spring AI inside the same monolith under the `ai` feature.
7. Start AI with read-only tools. AI must not directly mutate business-critical state.
8. React Native must not call the LLM provider (DeepSeek) directly. AI requests go through Spring Boot.

## 6. Technology Baseline

The backend versions below reflect the current `backend/pom.xml`, `backend/Dockerfile`, and Docker
Compose stack. The frontend remains a scaffold, so its library versions are intentionally not pinned
yet.

| Area | Technology or approach | Status |
|---|---|---|
| Backend runtime | Spring Boot 3.4.4, Java 21, Maven 3.9+ | Current scaffold |
| Backend API | Spring Web MVC, Validation, Springdoc OpenAPI 2.7.0, Actuator | Current scaffold |
| Backend mapping | MapStruct 1.6.3 with Lombok binding for entity/DTO mapping | Current backend dependency |
| Mobile runtime | React Native with Expo | Scaffold; version TBD |
| HTTP client | Axios | Planned frontend dependency |
| Server state | TanStack Query | Planned frontend dependency |
| Token storage | Expo SecureStore | Planned frontend dependency |
| Database | PostgreSQL 16 with pgvector image | Current local container scaffold |
| Persistence | Spring Data JPA/Hibernate | Current backend dependency |
| Database migrations | Flyway | Current backend dependency |
| Backend container image | Multi-stage `backend/Dockerfile` (Maven build stage + Temurin 21 JRE runtime, non-root user) | Current scaffold |
| Local orchestration | Docker Compose: PostgreSQL/pgvector + backend API, DB health-gated | Current scaffold |
| Configuration | Single runtime `application.yml`; DB and JWT values come from environment variables (optional `.env` import); REST controllers get a central `/api/v1` prefix | Current scaffold |
| Authentication | Spring Security with JJWT 0.12.6 | Current backend dependency; flow is phased |
| Human chat | Spring WebSocket and mobile WebSocket client | Backend dependency; feature phased |
| AI | Spring AI with an external LLM provider | Planned backend phase |
| Payment provider | VNPAY, with COD support | Planned feature |
| Testing | Spring Boot Test, JUnit 5, Testcontainers, H2 | Current backend scaffold |
| Money representation | `BigDecimal` | Backend implementation rule |

## 7. Backend Organization

The backend uses package-by-layer under `com.marketplace`. Each layer holds the classes of all
business features (`auth`, `user`, `catalog`, `cart`, `order`, `payment`, `shipment`, `voucher`,
`review`, `chat`, `ai`); a class name identifies its feature, for example `AuthController`,
`ProductService`, or `OrderRepository`.

```text
config | controller | service (+ impl) | repository | entity | enums
dto (request, response) | mapper | security | websocket | vnpay | ai (tools, rag, memory)
common (config, response, exception, util)
```

Classes shared by every feature (response envelope, shared exceptions, web configuration, helpers) live under `common`. Class names stay unique across the project because layers are shared. Features call each other only
through service interfaces, never through another feature's repository.

## 8. Software Development Plan

### Phase 0 - Project Foundation

- Initialize the Spring Boot monolith.
- Configure PostgreSQL.
- Use Flyway for versioned database migrations.
- Create the common error response.
- Add the global exception handler.
- Configure validation.
- Establish the package-by-layer convention.
- Add the base test setup.

**Completion gate:** the application starts, database migration runs, and `/actuator/health` is
healthy.

### Phase 1 - Auth and User

- Create the user entity and repository.
- Add roles.
- Implement register, login, and refresh.
- Add JWT access and refresh tokens.
- Encode passwords.
- Configure Spring Security authorization.
- Add the seller approval state.
- Add admin user management and seller approval, so a `PENDING` seller can be approved.
- Add password management (Phase 1 addition): change password, forgot password, and reset with a one-time code sent by email.

**Completion gate:** CUSTOMER, SELLER, and ADMIN login and authorization are verifiable by tests, an admin can approve a seller who then manages products, and a user can change their password and reset a forgotten one with an emailed code.

### Phase 2 - Catalog

- Add category CRUD for Admin.
- Add seller product CRUD.
- Add stock management.
- Add active or soft-delete product behavior.
- Add customer browse, search, filter, and pagination.

**Completion gate:** seller ownership and customer catalog flow are covered by integration tests.

### Phase 3 - Cart

- Add item creation, update, and removal.
- Keep one product unique per customer cart.
- Validate active product and stock.
- Recalculate price server-side.

**Completion gate:** invalid stock and inactive products cannot proceed.

### Phase 4 - Checkout and Order

- Create orders transactionally.
- Snapshot product name and price in order items.
- Split seller ownership at the order-item level.
- Apply vouchers.
- Calculate totals.
- Update stock.
- Set the prepayment deadline to 24 hours after order creation; COD follows the delivery collection rule.
- Complete COD only after all required non-cancelled seller shipments are delivered in the MVP; do not
  invent partial settlement for mixed seller outcomes.

**Completion gate:** a cart with products from at least two sellers creates one order with correctly
attributed order items.

### Phase 5 - Seller Order Processing

- Query seller order items.
- Confirm or cancel order items.
- Enforce ownership.
- Record cancellation reason if required.
- Do not restore shipment stock when delivery becomes `FAILED`.
- Restore the shipped quantity only after the shipment becomes `RETURNED`.

### Phase 6 - Payment

- Create COD payment records.
- Create VNPAY requests and process callbacks.
- Validate callback signatures.
- Make callbacks idempotent.
- Retry failed payments by creating another `Payment` record.
- Update order status to PAID according to payment rules.
- Cancel prepayment orders after 24 hours and restore checkout stock exactly once; COD follows the
  delivery collection rule.

### Phase 7 - Shipment

- Let the seller select platform delivery or self delivery.
- Create shipments only from valid confirmed items.
- Enforce one seller across all items in a shipment.
- Track shipment status transitions.
- Support `FAILED -> RETURNED`; restore stock only after `RETURNED`.
- Set the order to DELIVERED only after all required shipments are delivered.

### Phase 8 - Review and Voucher Administration

- Check customer review eligibility.
- Enforce one review per customer/product.
- Add Admin voucher CRUD.
- Validate voucher validity and usage.

### Phase 9 - Human Chat

- Pair customer and seller in a conversation.
- Persist messages.
- Add real-time WebSocket updates.
- Load offline message history.
- Add retry and idempotency handling for messages.

### Phase 10 - Spring AI

Implement AI in this order:

```text
ChatClient
 -> read-only tools
 -> chat memory
 -> RAG + pgvector
 -> semantic search/review summary
```

Do not enable AI write actions in the first implementation.

The current AI scope is unauthenticated/customer product search. Authenticated customer order and
shipment tools are later read-only work. Seller revenue analysis is planning only.

### Phase 11 - React Native Integration

- Integrate auth and navigation.
- Integrate catalog, cart, and checkout.
- Integrate orders, payment, and shipment tracking.
- Integrate reviews.
- Integrate human chat.
- Integrate the AI chat UI.

### Phase 12 - Hardening

- Add unit and integration tests.
- Add security ownership tests.
- Add payment idempotency tests.
- Add multi-seller order and shipment tests.
- Add AI tool authorization tests.
- Check product search and pagination performance.
- Keep the Docker Compose stack (PostgreSQL/pgvector plus backend image) and deployment docs current.
- Document production and test deployment.

## 9. Quality Gates

### 9.1 Minimum test scenarios

- A multi-seller order contains products from Seller A and Seller B.
- Seller A cannot confirm Seller B's order item.
- Seller A cannot put Seller B's item into a shipment.
- Cancelling Seller A's shipment does not incorrectly cancel Seller B's shipment.
- Seller A can be `DELIVERED` while Seller B is `FAILED` or `RETURNED` in the same order.
- `FAILED` does not restore stock; `RETURNED` restores only that seller's shipped quantity.
- A prepayment order is cancelled at 24 hours and restores checkout stock exactly once; COD is exempt.
- A COD order may move from `PENDING` directly to `DELIVERED`; its payment becomes `COMPLETED` after
  all required non-cancelled shipments are delivered.
- An order keeps the original product name and price after the product changes.
- A failed VNPAY attempt leaves the order intact.
- A retry creates a second payment record.
- A duplicate callback does not duplicate payment completion.
- Expired and exhausted vouchers are rejected.
- Concurrent use of the final voucher is transaction-safe.
- A customer who did not receive an item cannot review it.
- A delivered customer can review once.
- AI can access the current customer's order but not another customer's order.
- Guest AI can access only public product search and product information.
- AI uses RAG for policy questions and tools for live order/shipment data.
- AI cannot cancel, refund, change stock, or change price.

### 9.2 Definition of Done

A feature is complete only when:

- domain and business rules are implemented;
- DTO validation exists;
- authorization and ownership are enforced;
- relevant unit and integration tests pass;
- API errors are consistent;
- a database migration exists;
- the frontend can consume the API when it is part of the current milestone;
- business-critical rules do not exist only in the frontend;
- AI tools do not bypass normal application authorization.

## 10. Project-Level Architecture Target

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

The target remains one deployable monolith while keeping features internally separated enough to
remain maintainable and interview-friendly.
