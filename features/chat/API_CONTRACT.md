# Human Chat API Contract

## 1. Scope

This contract covers the `chat` (human chat) feature boundary: customer/seller conversations,
persisted messages, real-time updates, and history loading.

Planned/confirmed surface:

```text
WS  /ws/chat
GET /api/conversations
GET /api/conversations/{id}/messages
```

Predicted surface (not yet frozen; included for forward compatibility):

```text
POST /api/conversations                     (Predicted)
POST /api/conversations/{id}/messages       (Predicted)
```

Human chat is deferred from the first mobile milestone and the first MVP wire contract (see SRS
§2.1 and root §10); the WebSocket message payload shape is phased. Human chat is separate from AI
chat: different persistence model (`Conversation`/`Message` vs `AiChatSession`/`AiMessage`)
(`FR-CHAT-05`).

Common wire envelope (`ApiResponse<T>`), pagination shape (`PageResponse<T>`), data formats (UUID,
ISO-8601 UTC, zero-based `page`), and shared error codes/HTTP mapping are defined in
[`..\..\API_CONTRACT.md`](../../API_CONTRACT.md) (§2) and are not repeated here except for
endpoint-specific behavior.

Feature source: `features/chat/README.md`. No field outside that README is introduced below.

---

### WS /ws/chat

Real-time channel for human-chat updates.

- Actor/Auth: `CUSTOMER` or `SELLER`. JWT handshake required; the server resolves the participant
  identity from the authenticated principal and never trusts a client-supplied role or user ID.
- Purpose: realtime delivery of new `Message` events plus read-state updates (`FR-CHAT-03`).
- Persistence and history remain REST-owned (`FR-CHAT-02`); the socket is a transport, not the
  source of truth.
- Phased: connection setup, subscribe paths, heartbeat/reconnect, and exact STOMP/sub-protocol
  frames are not frozen in this contract.

Maps to: `FR-CHAT-01`, `FR-CHAT-03`, `FR-CHAT-05`.

---

### GET /api/conversations

List conversations owned by the caller.

- Actor/Auth: `CUSTOMER` or `SELLER`. Requires `Authorization: Bearer <access-token>`.
- Ownership: returns only rows where the caller is a participant — `customerId == principal.id`
  for `CUSTOMER`, `sellerId == principal.id` for `SELLER`. A caller must not see another
  customer's or another seller's conversation.
- `ADMIN` monitoring, if ever exposed, is not participant impersonation and is not part of this
  endpoint.

Success: `200 OK` with `ApiResponse<List<ConversationDto>>`. Each `data` element uses only feature
README fields:

```json
{
  "id": "conversation-uuid",
  "customerId": "customer-uuid",
  "sellerId": "seller-uuid",
  "productId": null,
  "orderId": null,
  "createdAt": "2026-10-12T10:30:00Z",
  "lastMessageAt": null
}
```

| Field | Meaning |
|---|---|
| `id` | Conversation identifier |
| `customerId` | Customer participant |
| `sellerId` | Seller participant |
| `productId` | Optional product context, else `null` |
| `orderId` | Optional order context, else `null` |
| `createdAt` | Time the conversation was opened |
| `lastMessageAt` | Optional timestamp for sorting; `null` when no message exists |

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Authenticated actor is neither `CUSTOMER` nor `SELLER` |

Maps to: `FR-CHAT-01`, `FR-CHAT-02`.

---

### GET /api/conversations/{id}/messages

Load persisted message history for one conversation (offline history loading).

- Actor/Auth: `CUSTOMER` or `SELLER`. Requires `Authorization: Bearer <access-token>`.
- Ownership: caller must be a participant of conversation `{id}`. Non-participants receive
  `FORBIDDEN` (or `RESOURCE_NOT_FOUND` when the conversation must not be disclosed).

Path parameters:

| Parameter | Type | Meaning |
|---|---|---|
| `id` | UUID string | Conversation identifier |

Query parameters (all optional):

| Parameter | Meaning |
|---|---|
| `page` | Zero-based page index; default `0` |
| `size` | Page size; default `20`, backend applies a maximum |

Success: `200 OK` with `ApiResponse<PageResponse<MessageDto>>`. Each `content` element uses only
feature README fields:

```json
{
  "id": "message-uuid",
  "conversationId": "conversation-uuid",
  "senderId": "user-uuid",
  "content": "Is this headphone still in stock?",
  "imageUrl": null,
  "isRead": false,
  "createdAt": "2026-10-12T10:31:00Z"
}
```

| Field | Meaning |
|---|---|
| `id` | Message identifier |
| `conversationId` | Parent conversation |
| `senderId` | Actual sender; checked against conversation membership |
| `content` | Text content |
| `imageUrl` | Optional image attachment reference, else `null` |
| `isRead` | Whether the intended recipient has read the message |
| `createdAt` | Creation time used for ordering |

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Caller is not a participant of the conversation |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Conversation does not exist (or is not visible) |
| `VALIDATION_ERROR` | `400 Bad Request` | Invalid `page`/`size` |

Maps to: `FR-CHAT-01`, `FR-CHAT-02`, `FR-CHAT-05`.

---

### POST /api/conversations — Predicted

Get-or-create a conversation for a customer/seller pair.

- Status: **Predicted** — not in the SRS planned interface list; included because the domain rule
  `unique (customer, seller)` requires a creation path.
- Actor/Auth: `CUSTOMER` or `SELLER`. Requires `Authorization: Bearer <access-token>`.
- Semantics: idempotent get-or-create on unique `(customer, seller)`. Repeating the same pair
  returns the existing conversation rather than creating a duplicate.
- Identity is resolved from the authentication principal; the request supplies only the counterpart
  and optional context. Arbitrary participant IDs from the client are not trusted.

Request body (raw JSON):

```json
{
  "sellerId": "seller-uuid",
  "productId": null,
  "orderId": null
}
```

| Field | Rule |
|---|---|
| `sellerId` / `customerId` | Counterpart participant ID (which one is required depends on caller role); the caller side is taken from the token |
| `productId` | Optional product context, else `null` |
| `orderId` | Optional order context, else `null` |

Success: `200 OK` (existing conversation) or `201 Created` (new conversation) with
`ApiResponse<ConversationDto>` (same shape as `GET /api/conversations` element).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Actor is neither `CUSTOMER` nor `SELLER` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Counterpart user / referenced product / order not visible |
| `VALIDATION_ERROR` | `400 Bad Request` | Missing counterpart ID or malformed context |

Maps to: `FR-CHAT-01` (Predicted creation path).

---

### POST /api/conversations/{id}/messages — Predicted

REST fallback for sending a message (used when the WebSocket is unavailable; retry path).

- Status: **Predicted** — REST fallback complementing `WS /ws/chat`.
- Actor/Auth: participant only (`CUSTOMER` or `SELLER` who belongs to conversation `{id}`).
  Requires `Authorization: Bearer <access-token>`.
- `senderId` is resolved from the authentication principal and checked against conversation
  membership.
- Idempotency: the client must send `Idempotency-Key: <client-generated-key>`; retries with the
  same key return the original message rather than creating a duplicate (`FR-CHAT-04`).

```http
POST /api/conversations/{id}/messages
Authorization: Bearer <access-token>
Content-Type: application/json
Idempotency-Key: client-generated-key
```

Request body (raw JSON):

```json
{
  "content": "Is this headphone still in stock?",
  "imageUrl": null
}
```

Success: `201 Created` with `ApiResponse<MessageDto>` (same shape as message-history element).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Caller is not a participant of the conversation |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Conversation does not exist (or is not visible) |
| `VALIDATION_ERROR` | `400 Bad Request` | Empty `content` with no `imageUrl`, or malformed body |

Maps to: `FR-CHAT-02`, `FR-CHAT-03`, `FR-CHAT-04`.
