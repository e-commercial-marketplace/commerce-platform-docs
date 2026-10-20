# AI API Contract

## 1. Scope

This contract covers the `ai` feature boundary: natural-language product search, read-only tool
calling, and persisted AI chat history for authenticated customers.

Planned surface:

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
- Guest requests may be stateless (`sessionId` is `null`). Persisted sessions/messages belong to
  authenticated customers only (`FR-AI-10`).
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
| `message` | string | Required natural-language request |
| `sessionId` | UUID string or `null` | Optional persisted session; `null` for stateless guest request |

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
  "sessionId": null
}
```

The answer must describe the returned list; the backend remains the source of truth for the actual
count and product data. The AI must not receive provider API keys, arbitrary user IDs, or
write-capability tools.

Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | `400 Bad Request` | Missing/empty `message` or malformed body |
| `FORBIDDEN` | `403 Forbidden` | Authenticated non-`CUSTOMER` actor requests protected scope |
| `INTERNAL_ERROR` | `500 Internal Server Error` | Unexpected provider/model failure (no secrets or stack traces leaked) |

Maps to: `FR-AI-01`, `FR-AI-02`, `FR-AI-03`, `FR-AI-04`, `FR-AI-06`, `NFR-AI-01`.

---

### GET /api/v1/ai/sessions

List persisted AI sessions owned by the caller.

- Actor/Auth: `CUSTOMER` only. Requires `Authorization: Bearer <access-token>`.
- Ownership: returns only sessions where `ai_chat_sessions.userId == principal.id`. The owner ID
  is never accepted from the request body. `SELLER`/`ADMIN` have no AI access in the current scope.

Success: `200 OK` with `ApiResponse<List<AiChatSessionResponse>>`. Each `data` element uses only
feature README fields:

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
- Ownership: session `{id}` must satisfy `ai_chat_sessions.userId == principal.id`; otherwise the
  request is rejected with `FORBIDDEN` (or `RESOURCE_NOT_FOUND` when the session must not be
  disclosed).

Path parameters:

| Parameter | Type | Meaning |
|---|---|---|
| `id` | UUID string | AI session identifier |

Success: `200 OK` with `ApiResponse<List<AiMessageResponse>>`. Each `data` element uses only feature
README fields:

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
| `FORBIDDEN` | `403 Forbidden` | Caller does not own the session, or actor is not `CUSTOMER` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Session does not exist (or is not visible to the caller) |

Maps to: `FR-AI-10`, `NFR-AI-01`.
