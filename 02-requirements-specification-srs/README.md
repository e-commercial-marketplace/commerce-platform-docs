# 02. Requirements Specification (SRS)

## 1. Document Control

| Field | Value |
|---|---|
| Document ID | DOC-02 |
| Document title | Requirements Specification (SRS) |
| Project | E-Commerce Marketplace |
| Primary source | [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md) |
| Domain model reference | [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml) |
| Status | Requirements baseline |

## 2. Purpose and Scope

This SRS defines the planned functional requirements, business rules, external interfaces, security
requirements, data requirements, and verification scenarios for the E-Commerce Marketplace.

The requirements below are limited to capabilities and constraints stated in `PLAN.md` and the
referenced class diagram. The screen references define MVP capabilities rather than pixel-level
visual requirements. Request/response details are defined in [`API_CONTRACT.md`](../API_CONTRACT.md).

### 2.1 First mobile MVP disposition

The first frontend milestone implements the customer shopping path. Backend domain rules remain
authoritative even when the corresponding seller/admin or after-sales screen is deferred.

| Area | First MVP decision | Deferred decision |
|---|---|---|
| Catalog | Public browse, search, category/price/rating filters, pagination, product detail | Seller/Admin catalog screens |
| Product model | Single product image and no color/size variants | `ProductVariant` and variant inventory |
| Authentication | Customer register/login/refresh before server-backed checkout | Seller approval and Admin UI |
| Cart | Add, update, remove, and server-side totals | Client-only guest cart synchronization |
| Checkout | Shipping-address snapshot, fixed `VND` currency, `0.00` shipping fee, and transactional order creation | Address-book management and dynamic shipping pricing |
| Payment | COD first; payment remains pending until all required seller shipments are delivered | VNPAY request/callback integration and partial COD settlement |
| Voucher | No voucher UI or checkout application in the first slice | Voucher validation and concurrency-safe consumption |
| Review | One eligible customer review with rating and comment | Review tags and image uploads |
| AI | Public/customer product search through `POST /api/v1/ai/ask`; guest requests may be stateless | Order/shipment tools, RAG, persisted AI chat, seller revenue analysis |
| Delivery | Backend shipment state and stock rules are required | Customer tracking UI and Seller shipment screens |
| Human chat | Not part of the first mobile milestone | WebSocket conversations |

The MVP API shapes and common error contract are defined in [`API_CONTRACT.md`](../API_CONTRACT.md).

## 3. Product Perspective

The system is a modular monolith composed of a React Native (Expo) client and a Spring Boot backend.
The backend communicates with PostgreSQL/pgvector, VNPAY, and an external LLM provider.

The backend is the authority for authorization, ownership, business rules, transactions, payment
callbacks, and AI tool access.

## 4. Actors and User Classes

| Actor | Responsibilities |
|---|---|
| UNAUTHENTICATED USER | Browse public catalog data and use AI for public product search only |
| CUSTOMER | Browse products, manage cart, place orders, pay, track shipments, review products, chat, use AI assistance |
| SELLER | Manage owned products and stock, process owned order items, choose delivery, manage owned shipments, chat |
| ADMIN | Manage users/categories/vouchers, moderate violations, monitor operations, configure platform delivery, approve administrative actions such as refunds |
| Payment provider | Return VNPAY payment callback data |
| LLM provider | Provide model responses through the backend AI module |

## 5. Functional Requirements

### 5.1 Authentication and user management

| ID | Requirement |
|---|---|
| FR-AUTH-01 | The system shall support customer, seller, and admin roles. |
| FR-AUTH-02 | The system shall support register, login, refresh, and logout operations. |
| FR-AUTH-03 | The system shall use password hashing through Spring Security `PasswordEncoder`; plaintext passwords shall not be stored. |
| FR-AUTH-04 | The system shall use JWT access and refresh flow. |
| FR-AUTH-05 | The system shall apply seller approval states: `PENDING`, `APPROVED`, `REJECTED`, and `SUSPENDED`. Only `APPROVED` sellers may manage products. |
| FR-AUTH-06 | The system shall let an admin list, view, create, update, and disable users, and filter the list by role and search text. Disabling an account keeps its data and blocks login and refresh. |
| FR-AUTH-07 | The system shall let an admin approve, suspend, or reject sellers. An admin shall not be able to disable or demote their own account. |
| FR-AUTH-08 | The system shall let a signed-in user change their password by giving the current password. A successful change shall revoke all of the user's refresh tokens and return a new token pair. |
| FR-AUTH-09 | The system shall let a user who forgot their password request a one-time 6-digit code by email and set a new password with it. The request shall not reveal whether the email is registered. A code shall expire after 15 minutes, allow at most 5 wrong attempts, and be usable once. A successful reset shall revoke all of the user's refresh tokens. |
| FR-AUTH-10 | The system shall send email through a provider configured by environment variables. An email failure shall not reveal to the client whether an account exists. |
| FR-AUTH-11 | The system shall let a signed-in user view and update their own name, phone number, and address, and let an admin update them for any user. The phone number shall be a Vietnamese mobile number. |
| FR-AUTH-12 | The system shall let a signed-in user (customer, seller, or admin) log out by revoking their own refresh token. A refresh token that is unknown, already revoked, or owned by another user shall be rejected with the same error. Logout does not revoke the access token, which expires on its own. |
| FR-AUTH-13 | The system shall let a user sign in with a Google account by sending a Google ID token. The system shall verify the token's signature, issuer, audience, and expiry, shall require an email verified by Google, and shall never trust an email sent separately by the client. An account found by Google subject or by email is signed in and linked to Google; otherwise a `CUSTOMER` account is created. A disabled account shall be rejected, and the response shall be the same token pair as login. |

FR-AUTH-08 to FR-AUTH-11 were added to Phase 1. FR-AUTH-12 describes the logout already implemented in Phase 1. FR-AUTH-13 is planned and not yet implemented.

Planned endpoints:

```text
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
POST /api/v1/auth/google

GET    /api/v1/users/me
PATCH  /api/v1/users/me
GET    /api/v1/admin/users
POST   /api/v1/admin/users
GET    /api/v1/admin/users/{id}
PATCH  /api/v1/admin/users/{id}
DELETE /api/v1/admin/users/{id}
PATCH  /api/v1/admin/sellers/{id}/approve
PATCH  /api/v1/admin/sellers/{id}/suspend

POST   /api/v1/auth/password/forgot
POST   /api/v1/auth/password/reset
PATCH  /api/v1/users/me/password
```

### 5.2 Catalog

| ID | Requirement |
|---|---|
| FR-CAT-01 | Customers shall be able to browse products. |
| FR-CAT-02 | Customers shall be able to search, filter, and paginate products. |
| FR-CAT-03 | Customers shall be able to view a product and categories. |
| FR-CAT-04 | Sellers shall be able to create, update, soft-delete/unlist, and update stock for their own products. |
| FR-CAT-05 | An active product shall belong to exactly one category and one seller. |
| FR-CAT-06 | A seller shall not modify a product owned by another seller. |
| FR-CAT-07 | Customers shall be able to view a seller's shop (the shop name is the seller's name) and the seller's listed products. |
| FR-CAT-08 | Sellers shall be able to list and view their own products, including unlisted ones. |

Planned endpoints:

```text
GET    /api/v1/products
GET    /api/v1/products/{id}
GET    /api/v1/categories
GET    /api/v1/sellers/{id}

GET    /api/v1/seller/products
GET    /api/v1/seller/products/{id}
POST   /api/v1/seller/products
PATCH  /api/v1/seller/products/{id}
DELETE /api/v1/seller/products/{id}
PATCH  /api/v1/seller/products/{id}/stock
```

### 5.3 Cart

| ID | Requirement |
|---|---|
| FR-CART-01 | A customer shall be able to view the cart. |
| FR-CART-02 | A customer shall be able to add, update, and remove cart items. |
| FR-CART-03 | A customer cart shall contain at most one row for a product. |
| FR-CART-04 | The system shall validate that a product is active and sufficiently stocked before checkout. |
| FR-CART-05 | The backend shall recalculate price server-side. |

Planned endpoints:

```text
GET    /api/v1/cart
POST   /api/v1/cart/items
PATCH  /api/v1/cart/items/{id}
DELETE /api/v1/cart/items/{id}
```

### 5.4 Checkout and order

| ID | Requirement |
|---|---|
| FR-ORD-01 | The system shall create an order transactionally from the valid cart items the customer selected. |
| FR-ORD-02 | An order shall contain at least one order item. |
| FR-ORD-03 | The system shall snapshot product name and unit price in each order item at checkout. |
| FR-ORD-04 | The system shall reserve or decrease stock during checkout. |
| FR-ORD-05 | The system shall validate and apply a voucher according to its validity and usage rules (delivered in Phase 8). |
| FR-ORD-06 | The system shall calculate and save discount, shipping, and total snapshots. |
| FR-ORD-07 | The system shall remove the purchased cart items after successful checkout; unselected items stay in the cart. |
| FR-ORD-08 | One order may contain order items from multiple sellers. |
| FR-ORD-09 | A customer shall be able to list and view only their own orders. |
| FR-ORD-10 | The system shall cancel a prepayment order 24 hours after `Order.createdAt` when payment is incomplete and restore its checkout stock exactly once; COD orders have no prepayment deadline. |
| FR-ORD-11 | Direct order-item cancellation is allowed only before shipment assignment. An assigned `PENDING`/`PREPARING` shipment may be cancelled atomically, marking its mapped lines `CANCELLED` and releasing unshipped stock exactly once; cancellation after `SHIPPED` or later is rejected. |
| FR-ORD-12 | Before payment completion, a valid cancellation shall recalculate the order payable total and every pending payment amount from the remaining order-item snapshots; if all lines are cancelled, the order shall become `CANCELLED`. |

Planned endpoints:

```text
POST /api/v1/orders
GET  /api/v1/orders
GET  /api/v1/orders/{id}
```

### 5.5 Seller order processing

| ID | Requirement |
|---|---|
| FR-FUL-01 | A seller shall be able to query their own order items. |
| FR-FUL-02 | A seller shall be able to confirm or cancel their own order items. |
| FR-FUL-03 | Seller operations shall enforce `orderItem.seller.id == authenticatedSeller.id`. |
| FR-FUL-04 | Seller cancellation shall not affect another seller's order item or shipment. |
| FR-FUL-05 | A seller shall confirm an order item of a VNPAY order only after the order is `PAID`. Lines of an unpaid VNPAY order stay visible but cannot be confirmed; COD orders are not affected. |

Planned endpoints:

```text
GET   /api/v1/seller/order-items
PATCH /api/v1/seller/order-items/{id}/confirm
PATCH /api/v1/seller/order-items/{id}/cancel
```

### 5.6 Payment

| ID | Requirement |
|---|---|
| FR-PAY-01 | The system shall support COD and VNPAY. |
| FR-PAY-02 | A COD payment shall start as `PENDING` and complete only after all required non-cancelled seller shipments are `DELIVERED`; partial settlement for mixed `FAILED`/`RETURNED` outcomes is deferred. |
| FR-PAY-03 | A VNPAY payment shall be created as `PENDING`, sent to the provider, and completed or failed by callback. |
| FR-PAY-04 | The system shall validate VNPAY callback signatures/checksums. |
| FR-PAY-05 | Payment callbacks shall be idempotent. |
| FR-PAY-06 | A failed VNPAY attempt shall not destroy the order. A retry shall create another `Payment` record. |
| FR-PAY-07 | VNPAY transaction/reference identifiers shall be unique where applicable. |
| FR-PAY-08 | VNPAY results arriving through the IPN and through the Return URL shall be processed by the same signature-checked, idempotent handler, so a payment is recorded whichever arrives first. A retry or a change of the order total shall mark every pending VNPAY attempt `FAILED` before a new payment link is issued. |

Planned endpoints:

```text
POST /api/v1/payments/vnpay/create
GET  /api/v1/payments/vnpay/ipn
GET  /api/v1/payments/vnpay/return
```

### 5.7 Shipment and delivery

| ID | Requirement |
|---|---|
| FR-SHIP-01 | A seller shall choose `PLATFORM_DELIVERY` or `SELF_DELIVERY`. A `PLATFORM_DELIVERY` shipment shall use one active carrier from the list managed by admin; a `SELF_DELIVERY` shipment has no carrier and is chosen at the seller's discretion. |
| FR-SHIP-02 | A shipment shall contain order items from exactly one seller. |
| FR-SHIP-03 | A shipment may group multiple order items from the same seller. |
| FR-SHIP-04 | A shipment shall be created only from valid confirmed order items belonging to the same order and seller. |
| FR-SHIP-05 | When status becomes `SHIPPED`, `trackingCode` and `shippedAt` shall exist. |
| FR-SHIP-06 | When status becomes `DELIVERED`, `deliveredAt` shall exist. |
| FR-SHIP-07 | The order shall become `DELIVERED` only after all required shipments are delivered. |
| FR-SHIP-08 | A customer shall be able to query shipment information only for their own order. |
| FR-SHIP-09 | Each seller shipment shall report its own `DELIVERED`, `FAILED`, or `RETURNED` outcome. |
| FR-SHIP-10 | A multi-seller order with mixed shipment outcomes shall not be reported as order-level `DELIVERED`. |
| FR-SHIP-11 | A shipment marked `FAILED` shall not restore stock. |
| FR-SHIP-12 | A shipment may be marked `RETURNED` only after the package reaches the seller warehouse; only then shall its shipped quantity be restored. |
| FR-SHIP-13 | In the MVP, each order item may belong to at most one shipment and shipment mapping uses the complete order-item line; partial-line, split, and replacement shipments are deferred. |
| FR-SHIP-14 | Admin shall add, rename, activate and deactivate carriers. Sellers shall see only active carriers. A carrier is never deleted; a deactivated carrier stays on the shipments that used it. |

Planned endpoints:

```text
POST  /api/v1/seller/shipments
GET   /api/v1/seller/shipments
PATCH /api/v1/seller/shipments/{id}/status
GET   /api/v1/seller/carriers
GET   /api/v1/admin/carriers
POST  /api/v1/admin/carriers
PATCH /api/v1/admin/carriers/{id}
```

### 5.8 Vouchers

| ID | Requirement |
|---|---|
| FR-VCH-01 | An order shall use at most one voucher. |
| FR-VCH-02 | The system shall validate voucher validity and usage. |
| FR-VCH-03 | The system shall reject expired or exhausted vouchers. |
| FR-VCH-04 | Voucher consumption shall use an atomic conditional update or equivalent optimistic-locking strategy so concurrent requests cannot exceed `usageLimit`. |

Planned endpoints:

```text
POST /api/v1/orders/{id}/voucher
GET  /api/v1/vouchers/{code}/validate
```

### 5.9 Reviews

| ID | Requirement |
|---|---|
| FR-REV-01 | Only a customer who purchased and received a product shall be eligible to review it. |
| FR-REV-02 | A customer shall have at most one review per product. |
| FR-REV-03 | A review rating shall be within the range 1 to 5. |

Planned endpoints:

```text
POST /api/v1/products/{productId}/reviews
GET  /api/v1/products/{productId}/reviews
```

### 5.10 Human chat

| ID | Requirement |
|---|---|
| FR-CHAT-01 | The system shall support customer/seller conversations. |
| FR-CHAT-02 | The system shall persist messages and support offline history loading. |
| FR-CHAT-03 | The system shall provide real-time updates through WebSocket. |
| FR-CHAT-04 | The system shall provide a retry/idempotency strategy for messages. |
| FR-CHAT-05 | Human chat shall remain separate from AI chat. |

Planned interfaces:

```text
WS  /ws/chat
GET /api/v1/conversations
GET /api/v1/conversations/{id}/messages
```

### 5.11 AI assistant

| ID | Requirement |
|---|---|
| FR-AI-01 | The AI assistant shall run inside the Spring Boot monolith through Spring AI. |
| FR-AI-02 | The system shall provide `POST /api/v1/ai/ask`. |
| FR-AI-03 | The first AI implementation shall use read-only tools. |
| FR-AI-04 | Protected AI tools shall resolve the authenticated customer from Spring Security context; guest requests shall be limited to public catalog data. |
| FR-AI-05 | AI tools shall call existing application services rather than bypassing them with direct repository access unless strongly justified. |
| FR-AI-06 | The initial AI capability shall map natural-language requests to a constrained product-search query, call the catalog service, and generate an answer grounded in returned products. |
| FR-AI-07 | Later authenticated-customer phases may expose read-only current-order, shipment, tracking, and voucher operations. |
| FR-AI-08 | The system shall not provide initial AI tools for cancelling orders, refunding payments, changing prices, updating stock, marking delivery, or deleting reviews. |
| FR-AI-09 | RAG shall be used for semi-static policies and FAQs; application services shall be used for live catalog and customer data. |
| FR-AI-10 | Persisted AI sessions and messages shall belong to authenticated customers; guest product-search requests may be stateless. |
| FR-AI-11 | Seller revenue analysis is planning only and shall not be enabled for the current AI scope. |

Planned endpoints:

```text
POST /api/v1/ai/ask                         (public catalog search; CUSTOMER protected scope)
GET  /api/v1/ai/sessions                    (CUSTOMER only)
GET  /api/v1/ai/sessions/{id}/messages      (CUSTOMER and own session only)
```

## 6. Business Rules and Domain Invariants

The following rules apply across features:

- A product belongs to exactly one category and one seller.
- A seller can modify only products they own.
- A shipment seller must equal the seller of every order item in that shipment.
- A shipment contains items from exactly one seller.
- `shipment.orderId == orderItem.orderId` for every shipment mapping.
- The MVP uses at most one shipment per seller within an order and whole order-item lines only.
- `OrderItemStatus` tracks seller processing; shipment progress belongs to `ShipmentStatus`.
- `OrderStatus` is an order-level aggregate; `FAILED` and `RETURNED` are seller-scoped `ShipmentStatus` values.
- One order may have multiple payment attempts.
- Money is represented as `VND`; the first MVP uses a fixed `0.00` shipping fee.
- One order uses at most one voucher.
- Human chat and AI chat are separate concepts.
- AI must not perform irreversible business actions in the initial implementation.

The referenced class diagram additionally defines the following model constraints:

- `sellerStatus` is required only when `role = SELLER`.
- Product `price > 0`, `stock >= 0`, and `active = false` means soft-deleted or unlisted.
- `transactionId` is required and unique for VNPAY.
- Cart uniqueness is `(user, product)` and `quantity >= 1`.
- Review uniqueness is `(user, product)` and rating is in `[1..5]`.
- `trackingCode` and `shippedAt` are required when a shipment is `SHIPPED`.
- `deliveredAt` is required when a shipment is `DELIVERED`.
- `sellerStatus` is nullable unless `role = SELLER`; optional payment, shipment, chat, and AI fields
  remain nullable until their state requires a value.
- The MVP human chat rule is unique `(customer, seller)`.

## 7. Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-ARCH-01 | The backend shall remain one modular monolith and one deployable unit. |
| NFR-ARCH-02 | Features shall communicate through normal Spring bean calls inside the JVM. |
| NFR-SEC-01 | JWT access/refresh flow shall be used for authentication. |
| NFR-SEC-02 | Role checks and ownership checks shall both be enforced. |
| NFR-SEC-03 | The system shall not trust role or user IDs supplied by React Native. |
| NFR-SEC-04 | Provider API keys shall not be exposed to React Native. |
| NFR-DATA-01 | Money shall use `BigDecimal`. |
| NFR-DATA-02 | Consistency-sensitive flows shall use PostgreSQL transactions. |
| NFR-DATA-03 | Database changes shall be managed by Flyway. |
| NFR-API-01 | API errors shall use a consistent response contract. |
| NFR-API-02 | Product search and pagination shall receive performance checks during hardening. |
| NFR-API-03 | Production responses shall not expose internal exception details or framework binding messages; unexpected failures shall use `INTERNAL_ERROR`. |
| NFR-AI-01 | Protected AI tools shall resolve the authenticated customer and respect normal application authorization; guest access is public-catalog-only. |
| NFR-AI-02 | RAG shall not be used as the source of truth for live transactional data. |

## 8. External Interface Requirements

### 8.1 Mobile client

React Native shall use the backend for business operations. The planned client data layer may use
Axios for HTTP, TanStack Query for server state, SecureStore for tokens, and a WebSocket/STOMP or
chosen WebSocket client for human chat.

The frontend may hide inaccessible actions, but authorization shall still be enforced by Spring Boot.

### 8.2 Payment provider

VNPAY integration shall support request creation, redirect, callback handling, signature/checksum
validation, unique transaction/reference identifiers, and idempotent completion.

### 8.3 LLM provider

The selected DeepSeek provider (OpenAI-compatible API) shall be configured on the backend. The mobile client
shall never call the provider directly.

### 8.4 Database and vector store

PostgreSQL stores marketplace data. pgvector stores embeddings for the RAG phase. Live transactional
data shall be read through application services/tools rather than treated as static RAG content.

### 8.5 Email provider

The backend shall send transactional email, such as password reset codes, through SMTP configured
by environment variables. Development uses Gmail SMTP with an App Password; the production provider
is not selected yet. When email is turned off for local
development, the code is written to the application log instead; production shall turn email on.

## 9. Verification and Acceptance

Each feature is accepted only when its business rules, DTO validation, authorization/ownership,
consistent errors, database migration, and relevant tests are complete. Frontend consumption is also
required when the feature belongs to the current frontend milestone.

The minimum verification scenarios are listed in `PLAN.md` and are summarized in section 9.1 of
[`01-project-overview-and-sdp/README.md`](../01-project-overview-and-sdp/README.md).

Current frontend automated verification is documented in
[`04-testing/README.md`](../04-testing/README.md). That document records the automated frontend tests
that exist for the current milestone. It is **not** evidence that every requirement in this SRS has
already passed acceptance testing; features that are not implemented remain unverified.
