# MVP API Contract

## 1. Purpose and scope

This document defines the minimum request and response contract for the first customer-facing mobile
MVP. It is the shared boundary between the React Native client and the Spring Boot backend.

The contract covers:

- customer authentication;
- public catalog and product search;
- cart management;
- checkout and COD order creation;
- customer order lookup;
- basic product reviews;
- guest/customer AI product search.

Seller/Admin screens, human chat, voucher application, VNPAY integration, advanced shipment tracking,
product variants, and review attachments are deferred from this contract's first implementation.

## 2. Common conventions

### 2.1 Base path and authentication

All business endpoints use the `/api/v1` base path. The version prefix is added centrally (`api.prefix`
in `application.yml`, applied by `WebConfig`), so controllers declare routes without it; operational
paths (actuator, OpenAPI, Swagger UI) stay unversioned. Protected requests send:

```http
Authorization: Bearer <access-token>
```

Public catalog (including seller shop pages), guest AI product-search, and review-list requests do not require a token. Cart,
checkout, order, and review-creation operations require an authenticated `CUSTOMER`.

### 2.2 Data formats

| Value | Format |
|---|---|
| Identifier | UUID string |
| Timestamp | ISO-8601 UTC string, for example `2026-10-12T10:30:00Z` |
| Money | Decimal string in `VND`, for example `"2800.00"`, to preserve precision across Java and JavaScript |
| Page index | Zero-based integer |
| Page size | Positive integer; backend applies a maximum |
| Empty optional value | `null`, not an empty string |

### 2.3 Wire envelope and error response

Every top-level endpoint response that contains a body uses `ApiResponse<T>` from the `common` module.
`204 No Content` responses have no body. Request bodies remain raw JSON. Response examples below
include the complete wire envelope unless they are explicitly labelled as a nested data payload. The
endpoint-specific result is always inside `data`.

Successful responses use this envelope:

```json
{
  "success": true,
  "data": "<endpoint-specific payload>",
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Every rejected request uses the same shape:

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "fieldErrors": {
      "quantity": "Must be greater than zero"
    }
  },
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

`fieldErrors` is optional and may be `null`. The backend must not expose framework binding details,
stack traces, provider secrets, or SQL errors. A request to a route that does not exist returns
`404 Not Found` with error code `RESOURCE_NOT_FOUND`.

Initial error codes:

```text
VALIDATION_ERROR
AUTHENTICATION_REQUIRED
FORBIDDEN
RESOURCE_NOT_FOUND
PRODUCT_INACTIVE
INSUFFICIENT_STOCK
EMPTY_CART
ORDER_NOT_PAYABLE
CART_ITEM_NOT_FOUND
CANNOT_CANCEL_AFTER_SHIPMENT
CANNOT_CANCEL_AFTER_PAYMENT
REVIEW_NOT_ELIGIBLE
REVIEW_ALREADY_EXISTS
VOUCHER_EXPIRED
VOUCHER_EXHAUSTED
VOUCHER_ALREADY_APPLIED
PAYMENT_FAILED
AI_UNAVAILABLE
INTERNAL_ERROR
```

HTTP mapping for the first MVP:

| Status | Use |
|---|---|
| `200 OK` | Successful read or mutation that returns a body |
| `201 Created` | Successful resource creation, such as registration or order creation |
| `204 No Content` | Successful deletion when the endpoint returns no body |
| `400 Bad Request` | Malformed JSON, validation failure, or invalid state transition |
| `401 Unauthorized` | Missing, expired, or invalid authentication |
| `403 Forbidden` | Authenticated actor lacks the required role or ownership |
| `404 Not Found` | Resource does not exist or is not visible to the caller |
| `409 Conflict` | Business conflict such as insufficient stock, duplicate review, or an order that is no longer payable |
| `500 Internal Server Error` | Unexpected server failure mapped to `INTERNAL_ERROR` |

## 3. Authentication

### Register

```http
POST /api/v1/auth/register
Content-Type: application/json
```

```json
{
  "email": "customer@example.com",
  "password": "strong-password",
  "name": "Customer",
  "role": "CUSTOMER"
}
```

`role` is optional and defaults to `CUSTOMER`. Public registration accepts only `CUSTOMER` or
`SELLER` (a seller application starts with `PENDING` status and requires admin approval); it must
never create an `ADMIN` account.

### Login

```http
POST /api/v1/auth/login
Content-Type: application/json
```

```json
{
  "email": "customer@example.com",
  "password": "strong-password"
}
```

### Auth response

```json
{
  "success": true,
  "data": {
    "accessToken": "jwt-access-token",
    "refreshToken": "jwt-refresh-token",
    "expiresIn": 900,
    "user": {
      "id": "user-uuid",
      "email": "customer@example.com",
      "name": "Customer",
      "role": "CUSTOMER"
    }
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

The three authentication endpoints return `ApiResponse<AuthResponse>` on success. Invalid
credentials return `401` with the common error shape. The client must use the token pair returned by
refresh.

### Auth response data shape

The following code block shows only the `data` object inside the auth response; it is not a complete
wire response:

```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token",
  "expiresIn": 900,
  "user": {
    "id": "user-uuid",
    "email": "customer@example.com",
    "name": "Customer",
    "role": "CUSTOMER"
  }
}
```

```http
POST /api/v1/auth/refresh
Content-Type: application/json
```

```json
{
  "refreshToken": "jwt-refresh-token"
}
```

Refresh rotates the token pair: a successful refresh returns a new access/refresh pair and revokes
the used refresh token. Refresh tokens expire per backend configuration and can be revoked
(server-side) before expiry. The client must use the token pair returned by refresh. Rotation and
revocation storage (`refresh_tokens`, migration `V12`) is implemented in Phase 1.

```http
POST /api/v1/auth/logout
Authorization: Bearer <access-token>
Content-Type: application/json
```

```json
{
  "refreshToken": "jwt-refresh-token"
}
```

Logout revokes the given refresh token. After logout the revoked token returns `401` with
`AUTHENTICATION_REQUIRED` when reused. The logout endpoint and revocation enforcement are
implemented in Phase 1.

```http
POST /api/v1/auth/google
Content-Type: application/json
```

```json
{
  "idToken": "google-id-token"
}
```

Planned. Google login: the client sends the Google ID token and the backend verifies it, then returns
the same `ApiResponse<AuthResponse>` as login. A first-time Google user gets a `CUSTOMER` account; an
existing account with the same Google-verified email is linked. An invalid token, an unverified email,
or a disabled account returns `401` with `AUTHENTICATION_REQUIRED`.

## 4. Catalog

### List products

```http
GET /api/v1/products?q=headphone&categoryId=category-uuid&minPrice=10&maxPrice=500&minRating=4&sort=relevance&page=0&size=20
```

All query parameters are optional:

| Parameter | Meaning |
|---|---|
| `q` | Text search against product name and description |
| `categoryId` | Category filter |
| `sellerId` | Seller (shop) filter: only that seller's listed products |
| `minPrice` / `maxPrice` | Inclusive price range |
| `minRating` | Minimum rating from `0` to `5` |
| `sort` | `relevance`, `price_asc`, `price_desc`, or `rating_desc` |
| `page` | Zero-based page index; default `0` |
| `size` | Page size; default `20` |

Grid/list view is a frontend presentation choice and is not sent to the backend.

The default sort is `relevance` when `q` is present and `rating_desc` otherwise. `relevance` is a
backend search score, followed by `ratingAverage DESC`, `reviewCount DESC`, and `id ASC` as deterministic
tie-breakers. Other sorts use `id ASC` as the final tie-breaker. A `popular` sort is deferred until a
separate popularity metric is defined.

The complete wire response is `ApiResponse<PageResponse<ProductSummaryResponse>>`:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": "product-uuid",
        "name": "Headphone",
        "description": "Wireless headphone",
        "imageUrl": "https://example.com/headphone.jpg",
        "price": "59.00",
        "stock": 12,
        "ratingAverage": "4.50",
        "reviewCount": 99,
        "active": true,
        "category": {
          "id": "category-uuid",
          "name": "Electronics"
        },
        "sellerId": "seller-uuid"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

### Categories

```http
GET /api/v1/categories
```

The complete wire response is `ApiResponse<List<CategorySummaryResponse>>`:

```json
{
  "success": true,
  "data": [
    {
      "id": "category-uuid",
      "name": "Electronics"
    }
  ],
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

### Product detail

```http
GET /api/v1/products/{productId}
```

The response uses the same product fields as the list response. The first MVP supports one
`imageUrl` and no color/size variant selection.

The actual wire response is `ApiResponse<ProductResponse>`:

```json
{
  "success": true,
  "data": {
    "id": "product-uuid",
    "name": "Headphone",
    "description": "Wireless headphone",
    "imageUrl": "https://example.com/headphone.jpg",
    "price": "59.00",
    "stock": 12,
    "ratingAverage": "4.50",
    "reviewCount": 99,
    "active": true,
    "category": {
      "id": "category-uuid",
      "name": "Electronics"
    },
    "sellerId": "seller-uuid"
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

### Seller shop

```http
GET /api/v1/sellers/{sellerId}
```

Public. The shop page of a seller; the shop name is the seller's `name`. The products of the shop are read with
`GET /api/v1/products?sellerId={sellerId}`. Returns `404 RESOURCE_NOT_FOUND` when the user does not exist, is
not a `SELLER`, or is not `APPROVED`.

The wire response is `ApiResponse<SellerShopResponse>`:

```json
{
  "success": true,
  "data": {
    "id": "seller-uuid",
    "name": "Seller Name",
    "productCount": 12
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

`productCount` counts the seller's listed (`active = true`) products.

## 5. Cart

All cart endpoints require a `CUSTOMER` token. The server resolves the cart owner from the token.

```http
GET /api/v1/cart
```

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "cart-item-uuid",
        "productId": "product-uuid",
        "name": "Headphone",
        "imageUrl": "https://example.com/headphone.jpg",
        "unitPrice": "59.00",
        "quantity": 2,
        "lineTotal": "118.00",
        "availableStock": 12,
        "active": true,
        "sellerId": "seller-uuid",
        "createdAt": "2026-10-12T10:30:00Z"
      }
    ],
    "subtotal": "118.00"
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

The complete wire response is `ApiResponse<CartResponse>`.

```http
POST /api/v1/cart/items
Content-Type: application/json
```

```json
{
  "productId": "product-uuid",
  "quantity": 1
}
```

```http
PATCH /api/v1/cart/items/{cartItemId}
Content-Type: application/json
```

```json
{
  "quantity": 2
}
```

```http
DELETE /api/v1/cart/items/{cartItemId}
```

The backend validates product activity and stock. The frontend must not treat a displayed price as
the authoritative checkout price. If the product is already in the customer's cart, `POST` increments
the existing quantity instead of creating a duplicate row. The combined quantity is validated against
current stock. Items whose product became inactive or out of stock stay in the cart with
`active` and `availableStock` set so the client can show them as unavailable; `subtotal` counts only
purchasable items, and items are listed newest first (`createdAt`). See
[features/cart/API_CONTRACT.md](features/cart/API_CONTRACT.md) for the full rules. Successful `POST`, `PATCH`, and `DELETE` mutations return
`ApiResponse<CartResponse>` with status `200` in the first MVP; `INSUFFICIENT_STOCK` or
`CART_ITEM_NOT_FOUND` is returned in the common error envelope when the mutation cannot be applied.

## 6. Checkout and orders

### Create order

The server reads the cart lines the authenticated customer selected (`cartItemIds`), validates
products and stock, snapshots product name/price, decreases stock, and removes the purchased lines
from the cart in one transaction. Unselected lines stay in the cart.

```http
POST /api/v1/orders
Authorization: Bearer <access-token>
Content-Type: application/json
```

```json
{
  "cartItemIds": ["cart-item-uuid-1", "cart-item-uuid-2"],
  "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
  "paymentMethod": "COD"
}
```

`paymentMethod` is `COD` or `VNPAY`. With `VNPAY`, checkout also creates the first VNPAY attempt and
returns its signed `paymentUrl`, which the client opens to pay; `paymentUrl` is `null` for COD and in
order lookups. A VNPAY order not paid within 24 hours is cancelled. Payment endpoints are defined in
`features/payment/API_CONTRACT.md`.

The complete wire response is `ApiResponse<OrderResponse>`:

```json
{
  "success": true,
  "data": {
    "id": "order-uuid",
    "status": "PENDING",
    "paymentStatus": "PENDING",
    "paymentMethod": "COD",
    "paymentUrl": null,
    "createdAt": "2026-10-12T10:30:00Z",
    "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
    "shippingFee": "0.00",
    "discountAmount": "0.00",
    "totalAmount": "118.00",
    "items": [
      {
        "id": "order-item-uuid",
        "productId": "product-uuid",
        "productName": "Headphone",
        "unitPrice": "59.00",
        "quantity": 2,
        "status": "PENDING",
        "sellerId": "seller-uuid"
      }
    ]
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

MVP money is in `VND`, represented as decimal strings. Shipping is fixed at `"0.00"` in the first
slice; the order total is calculated by the backend as item subtotal minus discount plus shipping.
The future prepayment deadline is derived from `createdAt + 24 hours`; it is not a new domain field in
the current class model. A prepayment order that is not completed by that deadline is cancelled and
its checkout stock is restored exactly once. COD has no prepayment deadline.

The product-detail `buy now` action has no separate checkout contract in the first MVP. It adds the
selected product and quantity to the authenticated customer's normal cart, then navigates to the
standard checkout flow with that line selected. The order endpoint buys only the selected cart lines;
see [features/order/API_CONTRACT.md](features/order/API_CONTRACT.md). Vouchers are applied from
Phase 8, so the first checkout has `discountAmount` `"0.00"`.

### Customer orders

```http
GET /api/v1/orders?page=0&size=20
GET /api/v1/orders/{orderId}
```

The backend returns only orders owned by the authenticated customer. Shipment outcomes remain
seller-scoped inside the order detail; a mixed seller result must not be collapsed into one false
order-level delivery status.

`GET /api/v1/orders` returns `ApiResponse<PageResponse<OrderSummaryResponse>>`. Each summary
contains `id`, `status`, `paymentStatus`, `paymentMethod`, `createdAt`, and `totalAmount`.
`GET /api/v1/orders/{orderId}` returns `ApiResponse<OrderResponse>`; it reuses the order-creation fields and adds
`shipments: ShipmentResponse[]`. The shipment response below is therefore part of the order detail, not a
standalone top-level response.

The order detail contains the following nested `ShipmentResponse` data shape. This is not a standalone
top-level response and must not be returned without the `ApiResponse<OrderResponse>` envelope:

```json
{
  "id": "shipment-uuid",
  "orderId": "order-uuid",
  "sellerId": "seller-uuid",
  "deliveryMethod": "SELF_DELIVERY",
  "status": "SHIPPED",
  "trackingCode": "TRACK-001",
  "shippingFee": "0.00",
  "shippedAt": "2026-10-13T09:00:00Z",
  "deliveredAt": null,
  "items": [
    {
      "orderItemId": "order-item-uuid",
      "quantity": 2
    }
  ]
}
```

The first MVP maps whole order-item lines only. The response `quantity` is the quantity from the
mapped `OrderItem`, not a partial mapping quantity. An order item belongs to at most one shipment, and
`shipment.orderId == orderItem.orderId`; partial-line quantities, split shipments, and replacement
shipments are deferred. A shipment must contain at least one item, and all mapped items must have the
same seller as `shipment.sellerId`.

## 7. Reviews

Reviews are publicly readable. Only an authenticated customer who purchased and received the product
may create one review for that product.

```http
GET /api/v1/products/{productId}/reviews?page=0&size=20
```

```http
POST /api/v1/products/{productId}/reviews
Authorization: Bearer <access-token>
Content-Type: application/json
```

```json
{
  "rating": 5,
  "comment": "Good product and fast delivery"
}
```

Review tags and image attachments are not part of the first MVP contract.

Successful review creation returns `201` with `ApiResponse<ReviewResponse>`:

```json
{
  "success": true,
  "data": {
    "id": "review-uuid",
    "productId": "product-uuid",
    "userId": "customer-uuid",
    "rating": 5,
    "comment": "Good product and fast delivery",
    "createdAt": "2026-10-14T10:30:00Z"
  },
  "error": null,
  "timestamp": "2026-10-14T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Review listing returns `ApiResponse<PageResponse<ReviewResponse>>`; its `data` contains `content`, `page`, `size`,
`totalElements`, and `totalPages`. An ineligible customer receives
`REVIEW_NOT_ELIGIBLE`; a second review for the same customer/product receives `REVIEW_ALREADY_EXISTS`.

## 8. AI product search and marketplace Q&A

The mobile client sends natural language to the backend. The backend maps the request to a constrained
catalog query (for product questions) or to RAG-grounded policy/FAQ knowledge (for marketplace
questions) and generates an answer grounded in those results. No login is required: guests may ask
about public data (products, FAQ, policies). Questions about orders, shipments, vouchers, or accounts
require an authenticated customer; the backend resolves identity from the security context and never
trusts a client-supplied user ID.

```http
POST /api/v1/ai/ask
Content-Type: application/json
```

```json
{
  "message": "Find me a black headphone under 2,500,000 VND with good reviews",
  "sessionId": null
}
```

```json
{
  "success": true,
  "data": {
    "answer": "I found 1 headphone under 2,500,000 VND with a rating of at least 4 stars.",
    "query": {
      "text": "black headphone",
      "maxPrice": "2500000.00",
      "minRating": "4"
    },
    "products": [
      {
        "id": "product-uuid",
        "name": "Black Wireless Headphone",
        "imageUrl": "https://example.com/black-headphone.jpg",
        "price": "89.00",
        "ratingAverage": "4.60",
        "reviewCount": 42,
        "sellerId": "seller-uuid"
      }
    ],
    "sessionId": null
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

The complete wire response is `ApiResponse<AiProductSearchResponse>`.

The answer must describe the returned list; the backend remains the source of truth for the actual
count and product data.

Guest requests may be stateless. The AI must not receive provider API keys, arbitrary user IDs, or
write-capability tools. Because the MVP currency is `VND`, foreign-currency input must be clarified or
rejected; the backend must not silently convert it without an explicit exchange-rate policy.

The AI feature must never crash the app. When the provider is not configured or the provider call
fails, the backend returns `503` with `AI_UNAVAILABLE` and the client shows a "not available" state.
The public AI endpoint is rate-limited to prevent abuse. Provider wiring lands in Phase 10; until
then the backend answers `AI_UNAVAILABLE` for every AI request.

## 9. State rules required by the backend

```text
Prepayment order: PENDING -> PAID -> DELIVERED
COD order:        PENDING -> DELIVERED
Prepayment order: PENDING -> CANCELLED after the prepayment deadline

Shipment: PENDING -> PREPARING -> SHIPPED -> DELIVERED
Shipment: SHIPPED -> FAILED -> RETURNED
Shipment: PENDING/PREPARING -> CANCELLED (before payment completion)
```

- `FAILED` does not restore stock.
- Only a successful `RETURNED` transition restores the shipped quantity for that seller.
- A multi-seller order may have independent shipment outcomes.
- COD is created as `PENDING` and is marked `COMPLETED` only after all required shipments are
  `DELIVERED`; the final required `DELIVERED` transition is the collection trigger handled by the
  shipment/payment services in one transaction. If any seller shipment becomes `FAILED` or `RETURNED`,
  no COD collection is recorded and the MVP does not perform partial settlement or refund.
- Direct order-item cancellation is allowed only before shipment assignment. For an assigned
  `PENDING`/`PREPARING` shipment, the seller must cancel the shipment instead; that atomic operation
  marks the mapped whole order-item lines `CANCELLED` and releases their unshipped stock exactly once.
  Cancellation after `SHIPPED` or any later shipment state is rejected. Shipment cancellation after a
  prepayment is completed is also rejected until a refund policy exists.
- `OrderStatus.DELIVERED` considers only non-cancelled order items and is reached only when all their
  required shipments are delivered. A COD order can therefore move from `PENDING` directly to
  `DELIVERED` while its payment moves from `PENDING` to `COMPLETED`.
- When seller shipments have mixed terminal outcomes, the order remains a container for the seller-
  scoped shipment results; clients must display `DELIVERED`, `FAILED`, or `RETURNED` per shipment and
  must not invent a new order-level success/failure enum.
- For a valid cancellation before payment completion, the order service recalculates `totalAmount` and
  the `amount` of every still-`PENDING` payment atomically from the remaining non-cancelled order-item
  snapshots plus the fixed shipping fee. It never rereads the current product price; cancelled line
  snapshots remain unchanged for history.
- After cancellation, aggregate state is reevaluated with a non-empty guard: if all order items are
  `CANCELLED`, the order becomes `CANCELLED` and no COD collection is recorded; if non-cancelled items
  remain and all of their required shipments are already `DELIVERED`, the order/payment may complete in
  the same cancellation transaction; otherwise the order remains pending.
- VNPAY results (IPN and Return URL) are signature-checked and idempotent by `vnp_TxnRef`.

## 10. Deferred contract

The following interfaces are deliberately not frozen for the first MVP:

- voucher validation and application;
- seller product/order/shipment management;
- Admin operations;
- human chat WebSocket messages;
- product variants and variant stock;
- review tags and image uploads;
- persisted AI sessions and customer order/shipment AI tools.
