# AI API Contract

## 1. Scope

This contract covers the `ai` feature boundary: natural-language product search, read-only tool
calling, and persisted AI chat history for authenticated customers.

Surface (Confirmed):

```text
POST /api/v1/ai/ask
GET  /api/v1/ai/sessions
GET  /api/v1/ai/sessions/{id}/messages
```

Authorization boundary (hard rules):

- AI tools are read-only in the initial implementation (`FR-AI-03`).
- Protected AI tools resolve the authenticated customer from the Spring Security context; the model
  and the client must not supply arbitrary protected `userId` values (`FR-AI-04`, `NFR-AI-01`).
- Guest (unauthenticated) access is public-catalog-only; guest product-search requests may be
  stateless and must not create a customer-owned session (`FR-AI-10`).
- The initial AI layer must not expose `cancelOrder`, `refundPayment`, `changePrice`,
  `updateStock`, `markDelivered`, or `deleteReview` (`FR-AI-08`).

Common wire envelope (`ApiResponse<T>`), data formats (UUID, ISO-8601 UTC, `VND` money as decimal
string), and shared error codes/HTTP mapping are defined in
[`..\..\API_CONTRACT.md`](../../API_CONTRACT.md) (§2) and are not repeated here except for
endpoint-specific behavior.

Feature source: `features/ai/README.md`. No field outside that README (plus the root-contract
`AiProductSearchResponse` shape for `POST /api/v1/ai/ask`) is introduced below.

---

### POST /api/v1/ai/ask

Map a natural-language request to a constrained catalog query and return a grounded answer.

- Actor/Auth: public for catalog search; no token required for guest product search. When a token
  is present it must be a `CUSTOMER` token and protected (customer) scope applies.
- Guest requests are stateless: `sessionId` must be `null` and nothing is stored. Persisted
  sessions/messages belong to authenticated customers only (`FR-AI-10`).
- Customer with `sessionId = null`: the backend creates a new session and returns its id. Customer with a
  `sessionId`: it must be one of the caller's sessions; the model then remembers the last 20 user and
  assistant messages of that session (chat memory). Every turn is stored in `ai_messages`; system
  prompts are not stored.
- Product data comes from a read-only `searchProducts` tool that calls the catalog service (top 5 listed
  products). `query` and `products` in the response come from that call, not from the model's text.
- Flow: natural-language request → constrained `ProductSearchQuery` → field/sort/page validation
  → catalog service search → grounded natural-language answer (`FR-AI-06`). The AI must not invent
  availability, price, or stock outside the catalog-service result.
- Because the MVP currency is `VND`, foreign-currency input must be clarified or rejected; the
  backend must not silently convert it without an explicit exchange-rate policy.

Request body (raw JSON):

```json
{
  "message": "Find me a black headphone under 2,500,000 VND with good reviews",
  "sessionId": null
}
```

| Field | Type | Rule |
|---|---|---|
| `message` | string | Required, not blank, at most 1000 characters |
| `sessionId` | UUID string or `null` | Customer: `null` starts a new session, otherwise one of the caller's sessions. Guest: must be `null` |

Success: `200 OK` with `ApiResponse<AiProductSearchResponse>`. The `data` object is:

```json
{
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
  "sessionId": null,
  "answeredBy": "AI"
}
```

| Field | Meaning |
|---|---|
| `answer` | Natural-language answer, grounded in `products` |
| `query` | The search actually run: `text`, `minPrice`, `maxPrice`, `minRating`, `sort` (each may be `null`); `null` when no search was needed |
| `products` | At most 5 listed products with the fields shown above; `[]` when there was no search |
| `sessionId` | The customer's session (new or given); `null` for guests |
| `answeredBy` | `AI` when the LLM answered; `RULES` when the backend answered without the LLM |

The answer must describe the returned list; the backend remains the source of truth for the actual
count and product data. The AI must not receive provider API keys, arbitrary user IDs, or
write-capability tools.

Answering without the LLM (`answeredBy = RULES`):

- Used when the provider is not configured (`AI_API_KEY` empty), when the provider call fails, or when
  the caller asks more than 20 questions in the current minute. The request still returns `200 OK`.
- Steps: (1) read price limits (`dưới 5 triệu`, `500k`, `từ 1tr đến 3tr`, `2.500.000 đ`), a minimum
  rating (`từ 4 sao`), a sort (`rẻ nhất`, `đắt nhất`, `đánh giá cao`) and a category name or part of
  one (`tai nghe` for `Tai nghe & Loa`) from the message; the rest is the search text. (2) A message
  with an FAQ keyword (greeting, ordering, payment, delivery, cancellation, voucher, review) and none of
  those filters gets a fixed answer that only restates rules already in this contract. (3) Otherwise
  search the catalog: the whole text, then pairs of adjacent words (single words are not tried: a
  Vietnamese syllable alone matches unrelated products); with products, a templated answer about them,
  without products a "nothing found" answer. (4) Nothing to search: a hint on how to ask.
- A foreign currency (`$`, `USD`) gets an answer that only VND is supported; nothing is converted.
- For customers, rule-based turns are stored in the session like AI turns.

Rate limit: counted per customer (user id) or per guest (client IP) in a one-minute window, kept in the
backend's memory. Questions 1 to 20 may use the LLM, 21 to 60 are answered by rules, from 61 on the
request is rejected.

Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | `400 Bad Request` | Missing/empty `message`, longer than 1000 characters, or malformed body |
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | A guest sends a `sessionId` |
| `FORBIDDEN` | `403 Forbidden` | The token belongs to a `SELLER` or `ADMIN` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | `sessionId` does not exist or belongs to another customer |
| `RATE_LIMITED` | `429 Too Many Requests` | More than 60 questions in the current minute |

Maps to: `FR-AI-01`, `FR-AI-02`, `FR-AI-03`, `FR-AI-04`, `FR-AI-06`, `FR-AI-10`, `NFR-AI-01`.

---

### GET /api/v1/ai/sessions

List persisted AI sessions owned by the caller.

- Actor/Auth: `CUSTOMER` only. Requires `Authorization: Bearer <access-token>`.
- Ownership: returns only sessions where `ai_chat_sessions.userId == principal.id`. The owner ID
  is never accepted from the request body. `SELLER`/`ADMIN` have no AI access in the current scope.

Success: `200 OK` with `ApiResponse<List<AiChatSessionResponse>>`, newest session first. Each `data`
element uses only feature README fields:

```json
{
  "id": "ai-session-uuid",
  "userId": "customer-uuid",
  "createdAt": "2026-10-12T10:30:00Z"
}
```

| Field | Meaning |
|---|---|
| `id` | Stable identifier for one persisted customer conversation |
| `userId` | Authenticated customer who owns the session |
| `createdAt` | Time the session was created |

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Authenticated actor is not `CUSTOMER` |

Maps to: `FR-AI-10`, `NFR-AI-01`.

---

### GET /api/v1/ai/sessions/{id}/messages

Load persisted turns for one caller-owned AI session.

- Actor/Auth: `CUSTOMER` only. Requires `Authorization: Bearer <access-token>`.
- Ownership: session `{id}` must satisfy `ai_chat_sessions.userId == principal.id`. A session of
  another customer returns `RESOURCE_NOT_FOUND`, the same as a missing one, so ids are not disclosed.

Path parameters:

| Parameter | Type | Meaning |
|---|---|---|
| `id` | UUID string | AI session identifier |

Success: `200 OK` with `ApiResponse<List<AiMessageResponse>>`, oldest message first. Each `data`
element uses only feature README fields:

```json
{
  "id": "ai-message-uuid",
  "sessionId": "ai-session-uuid",
  "role": "ASSISTANT",
  "content": "I found 1 headphone under 2,500,000 VND.",
  "confidence": null,
  "createdAt": "2026-10-12T10:31:00Z"
}
```

| Field | Meaning |
|---|---|
| `id` | Stable identifier for one conversation turn |
| `sessionId` | Parent `ai_chat_sessions` record |
| `role` | `USER`, `ASSISTANT`, or `SYSTEM` |
| `content` | User input or the grounded assistant/system message |
| `confidence` | Optional confidence metadata; not an authorization decision, else `null` |
| `createdAt` | Time the turn was persisted |

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Actor is not `CUSTOMER` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Session does not exist or belongs to another customer |

Maps to: `FR-AI-10`, `NFR-AI-01`.
