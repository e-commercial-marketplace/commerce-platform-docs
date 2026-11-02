# Product Requirements Document (PRD)

## 1. Product definition

The E-Commerce Marketplace is a multi-seller marketplace delivered as a modular monolith. Customers
can browse and purchase products from multiple sellers in one order. Sellers manage their own catalog,
stock, order items, and shipments. Admins manage platform-level operations.

The target system consists of a React Native (Expo) mobile client and a Spring Boot backend connected
to PostgreSQL and pgvector. Payment uses COD and VNPAY. AI assistance is provided through Spring AI
inside the backend.

### 1.1 MVP scope

The first mobile MVP is a customer-facing shopping flow. The provided screen references define the
capabilities to implement, not a pixel-perfect visual specification.

#### Delivery milestones

- **Minimum MVP (must achieve):** the customer shopping flow below. Seller/Admin screens are not
  required in the first mobile MVP, but the backend must already enforce their domain rules.
- **Full roadmap:** the 3-role app (`CUSTOMER`, `SELLER`, `ADMIN`) with seller catalog/stock/order/
  shipment management, admin user/category/voucher management, human chat, VNPAY integration, and
  the full AI slice.

#### In scope

- Public catalog browsing, search, category filtering, price/rating filtering, pagination, and grid/list presentation.
- Product detail with product information, rating summary, quantity selection, and add-to-cart/buy-now actions.
- Customer registration/login required before server-backed cart checkout.
- Public registration creates a `CUSTOMER` account by default. Seller onboarding is offered on the
  account-creation screen: a seller application creates a `SELLER` account with `PENDING` status.
  Public registration must never create an `ADMIN` account.
- A new seller request triggers a notification email; admin approval triggers a success email to
  the seller. Email sending is best-effort and must not roll back registration/approval if it fails.
- The first admin account is bootstrapped from environment configuration when no admin exists.
  Credentials are never hard-coded in the repo.
- Cart item add, update, remove, and server-side total calculation.
- Checkout with a shipping-address snapshot and COD payment. VNPAY remains the next payment slice.
- Order creation, payment result, and basic customer order retrieval.
- Product reviews with rating and comment only.
- Public/customer AI product search and marketplace Q&A through the existing search flow; guest
  searches may be stateless and do not require login. Guest AI is limited to public data (products,
  FAQ, policies); order/shipment/account questions require an authenticated customer.

#### Deferred from the first mobile MVP

- Seller and Admin screens.
- Human chat, Favorites, and full Account screens.
- Advanced shipment-tracking UI; shipment state transitions remain backend responsibilities.
- Voucher administration and voucher application UI.
- Product variants such as color/size, review tags, and review image uploads.
- VNPAY request/callback integration.
- AI order/shipment tools, RAG, persisted AI chat history, and seller revenue analysis.

The backend still has to enforce the domain rules that are not represented by a dedicated MVP screen:
multi-seller order ownership, 24-hour prepayment cancellation, independent seller shipments,
`FAILED`/`RETURNED` handling, and stock restoration only after `RETURNED`.

## 2. Product actors

| Actor | Product responsibilities |
|---|---|
| UNAUTHENTICATED USER | Browse public catalog data and use AI for public product search |
| CUSTOMER | Browse products, manage cart, place orders, pay, track shipments, review products, chat, and use AI assistance |
| SELLER | Manage owned products and stock, process owned order items, choose delivery method, manage owned shipments, and chat with customers |
| ADMIN | Manage users, seller approval, categories, vouchers, and carriers |

## 3. Product capabilities

### Catalog and cart

- Customers browse, search, filter, and paginate products.
- Sellers manage their own products and stock.
- Customers add, update, and remove cart items.
- A customer cart contains at most one row for each product.
- Product activity, stock, and server-side price are validated by the backend.
- The first MVP uses `VND` and a fixed `0.00` shipping fee; the backend calculates the final total.

### Multi-seller checkout and orders

- One order may contain products from multiple sellers.
- The backend validates active products, stock, vouchers, and prices.
- Product name and price are snapshotted in `OrderItem` at checkout.
- Stock, totals, voucher use, and cart clearing are part of one consistency-sensitive operation.
- An order requiring prepayment is cancelled 24 hours after creation if payment is not completed;
  stock reserved/decreased for that order is restored exactly once. COD orders follow the delivery
  collection rule and are exempt from this prepayment deadline.
- An order item can be directly cancelled only before shipment assignment. An assigned `PENDING` or
  `PREPARING` shipment can be cancelled atomically, which cancels its mapped lines and releases
  unshipped stock exactly once; cancellation after `SHIPPED` is rejected.
- Before payment completion, a valid cancellation recalculates the order payable total and any pending
  payment amount from the remaining line snapshots. If all lines are cancelled, the order is
  `CANCELLED` and no COD is collected; if delivered non-cancelled lines are all that remain, the
  order/payment is finalized in the same transaction.
- Sellers process only their own order items.

### Payment

- COD starts as a pending payment and completes only after all required seller shipments are delivered
  in the MVP. Partial COD settlement for mixed seller outcomes is deferred.
- VNPAY starts as a pending payment and finishes through a validated callback.
- A failed VNPAY attempt leaves the order intact; retry creates another payment record.
- Callback processing is idempotent.

### Delivery and shipment

- Sellers choose platform delivery or self delivery.
- A shipment contains items from exactly one seller.
- A multi-seller order may produce multiple shipments.
- Shipment state is separate from order-item processing state.
- Each seller shipment is reported independently: `DELIVERED`, `FAILED`, or `RETURNED`.
- `FAILED` does not restore stock. After the package reaches the seller warehouse, the shipment is
  marked `RETURNED`, and only then is that seller's shipped quantity restored.
- A shipment belongs to exactly one order and one seller. In the MVP, one order item is assigned to at
  most one shipment, and shipment mapping uses whole order-item lines; split and replacement shipments
  are deferred.
- An order becomes delivered only after all required shipments are delivered.
- Mixed seller outcomes are displayed from the shipment list; they are not collapsed into a false
  order-level `DELIVERED` result or a new order-level failure enum.

### Reviews and vouchers

- Only a customer who purchased and received a product may review it.
- A customer may review a product once.
- An order uses at most one voucher.
- Expired, exhausted, or otherwise invalid vouchers are rejected.

### Human chat and AI assistant

- Human customer/seller conversations persist messages and support real-time WebSocket updates.
- Human chat and AI chat are separate concepts.
- Unauthenticated users may use AI for public product search and product information.
- `CUSTOMER` may use product search and later read-only tools for their own orders, shipments,
  tracking, and vouchers.
- `SELLER` and `ADMIN` are outside the current AI scope. Seller revenue analysis is planning only.
- AI maps natural-language requests to a constrained product-search query, lets the catalog service
  perform the authoritative search, and generates answers from the returned products.
- AI uses RAG for semi-static policies and tools for live transactional data.
- AI does not initially cancel orders, refund payments, change prices, update stock, mark delivery, or
  delete reviews.

## 4. Product rules

- The product remains a single deployable modular monolith.
- Business-critical authorization is enforced by Spring Boot, not only by the mobile UI.
- Role checks and ownership checks are both mandatory.
- React Native never receives or uses the LLM provider API key.
- Money uses `BigDecimal`.
- Consistency-sensitive operations use PostgreSQL transactions.

## 5. Product boundaries

The following are not part of the initial target unless a later requirement explicitly adds them:

- microservices;
- API Gateway or service discovery;
- Feign or internal inter-service REST;
- Kafka, Saga, or distributed transactions;
- AI write operations that directly mutate business-critical state.

## 6. Product acceptance boundary

The product is considered ready for a feature milestone only when its domain rules, DTO validation,
authorization and ownership, consistent API errors, migration, relevant tests, and frontend consumption
are complete where the feature is part of the current frontend milestone.

Important acceptance scenarios include multi-seller isolation, independent shipment outcomes,
failed-payment retry, 24-hour prepayment-order cancellation and stock restoration, duplicate callback
handling, voucher concurrency, review eligibility, and guest/customer AI data boundaries.
