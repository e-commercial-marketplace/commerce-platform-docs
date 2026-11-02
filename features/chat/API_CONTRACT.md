# Human Chat API Contract

## 1. Scope

This contract covers the `chat` (human chat) feature boundary: customer/seller conversations,
persisted messages, real-time updates, and history loading.

Surface:

```text
WS    /ws/chat
GET   /api/v1/conversations
POST  /api/v1/conversations
GET   /api/v1/conversations/{id}/messages
POST  /api/v1/conversations/{id}/messages
PATCH /api/v1/conversations/{id}/read
```

Human chat is deferred from the first mobile milestone (see SRS §2.1); the backend implements the
surface above. Messages are sent over REST and pushed over the WebSocket. Human chat is separate from AI
chat: different persistence model (`Conversation`/`Message` vs `AiChatSession`/`AiMessage`)
(`FR-CHAT-05`).

Common wire envelope (`ApiResponse<T>`), pagination shape (`PageResponse<T>`), data formats (UUID,
ISO-8601 UTC, zero-based `page`), and shared error codes/HTTP mapping are defined in
[`..\..\API_CONTRACT.md`](../../API_CONTRACT.md) (§2) and are not repeated here except for
endpoint-specific behavior.

Feature source: `features/chat/README.md`. No field outside that README is introduced below.

---

### WS /ws/chat

Push-only real-time channel for human-chat updates (plain WebSocket, no STOMP).

- Handshake: `GET /ws/chat` (no `/api/v1` prefix) with `Authorization: Bearer <access-token>`. Only
  `CUSTOMER` and `SELLER` may connect; a missing or invalid token is rejected with `401`, another role
  with `403`. The server takes the user from the token and never trusts a client-supplied ID.
- The client sends nothing; text frames from the client are ignored. Sending happens through
  `POST /api/v1/conversations/{id}/messages`.
- Events are JSON text frames, sent only after the database transaction commits (`FR-CHAT-03`):

```json
{ "type": "MESSAGE_CREATED", "message": { "id": "message-uuid", "conversationId": "conversation-uuid", "senderId": "user-uuid", "content": "Is this headphone still in stock?", "imageUrl": null, "isRead": false, "createdAt": "2026-10-12T10:31:00Z" } }
```

```json
{ "type": "MESSAGES_READ", "conversationId": "conversation-uuid", "readerId": "user-uuid" }
```

| Event | Sent to | When |
|---|---|---|
| `MESSAGE_CREATED` | Both participants (every open connection, so the sender's other devices update too) | A new message is stored (not on an idempotent resend) |
| `MESSAGES_READ` | The other participant | `PATCH /conversations/{id}/read` marked at least one message read |

- Persistence and history remain REST-owned (`FR-CHAT-02`); the socket is a transport, not the source
  of truth. A client that reconnects reloads history over REST. Reconnect logic belongs to the client.
- Open connections are kept in memory: this fits one backend instance.

Maps to: `FR-CHAT-01`, `FR-CHAT-03`, `FR-CHAT-05`.

---

### GET /api/v1/conversations

List conversations owned by the caller.

- Actor/Auth: `CUSTOMER` or `SELLER`. Requires `Authorization: Bearer <access-token>`.
- Ownership: returns only rows where the caller is a participant — `customerId == principal.id`
  for `CUSTOMER`, `sellerId == principal.id` for `SELLER`. A caller must not see another
  customer's or another seller's conversation.
- Order: most recent `lastMessageAt` first; conversations without messages follow, newest
  `createdAt` first.
- `ADMIN` monitoring, if ever exposed, is not participant impersonation and is not part of this
  endpoint.

Success: `200 OK` with `ApiResponse<List<ConversationResponse>>`. Each `data` element uses only feature
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

### GET /api/v1/conversations/{id}/messages

Load persisted message history for one conversation (offline history loading).

- Actor/Auth: `CUSTOMER` or `SELLER`. Requires `Authorization: Bearer <access-token>`.
- Ownership: caller must be a participant of conversation `{id}`. Non-participants receive
  `RESOURCE_NOT_FOUND`, so conversation ids are not disclosed.
- Order: newest message first; the next page holds older messages.

Path parameters:

| Parameter | Type | Meaning |
|---|---|---|
| `id` | UUID string | Conversation identifier |

Query parameters (all optional):

| Parameter | Meaning |
|---|---|
| `page` | Zero-based page index; default `0` |
| `size` | Page size; default `20`, backend applies a maximum |

Success: `200 OK` with `ApiResponse<PageResponse<MessageResponse>>`. Each `content` element uses only
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
| `FORBIDDEN` | `403 Forbidden` | Authenticated actor is neither `CUSTOMER` nor `SELLER` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Conversation does not exist or the caller is not a participant |
| `VALIDATION_ERROR` | `400 Bad Request` | Invalid `page`/`size` |

Maps to: `FR-CHAT-01`, `FR-CHAT-02`, `FR-CHAT-05`.

---

### POST /api/v1/conversations

Get-or-create a conversation for a customer/seller pair.

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
| `sellerId` | Required when the caller is a `CUSTOMER`: an `APPROVED` seller |
| `customerId` | Required when the caller is a `SELLER`: a `CUSTOMER` user |
| `productId` | Optional product context, else `null`; must exist |
| `orderId` | Optional order context, else `null`; must belong to the customer of the pair |

The context fields are stored only when the conversation is created; an existing conversation keeps
its original context.

Success: `200 OK` (existing conversation) or `201 Created` (new conversation) with
`ApiResponse<ConversationResponse>` (same shape as `GET /api/v1/conversations` element).

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Actor is neither `CUSTOMER` nor `SELLER` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Counterpart is not an approved seller / not a customer, or the product / order is not visible |
| `VALIDATION_ERROR` | `400 Bad Request` | Missing counterpart ID or malformed context |

Maps to: `FR-CHAT-01`.

---

### POST /api/v1/conversations/{id}/messages

Send a message. This is the only send path; the WebSocket only pushes.

- Actor/Auth: participant only (`CUSTOMER` or `SELLER` who belongs to conversation `{id}`).
  Requires `Authorization: Bearer <access-token>`.
- `senderId` is resolved from the authentication principal and checked against conversation
  membership.
- Idempotency: the client must send `Idempotency-Key: <client-generated-key>` (1 to 100 characters,
  for example a UUID); retries with the same key return the original message rather than creating a
  duplicate, and push no new event (`FR-CHAT-04`). The key is unique per sender.
- Effects: the message is stored unread, `conversation.lastMessageAt` becomes its `createdAt`, and
  `MESSAGE_CREATED` is pushed after commit.

```http
POST /api/v1/conversations/{id}/messages
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

| Field | Rule |
|---|---|
| `content` | Text, at most 2000 characters; may be empty only when `imageUrl` is set |
| `imageUrl` | Optional image URL (at most 2048 characters) from the Cloudinary upload flow, see `features/upload/API_CONTRACT.md` |

Success: `201 Created` with `ApiResponse<MessageResponse>` (same shape as message-history element),
also for an idempotent resend.

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Authenticated actor is neither `CUSTOMER` nor `SELLER` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Conversation does not exist or the caller is not a participant |
| `VALIDATION_ERROR` | `400 Bad Request` | Missing `Idempotency-Key`, empty `content` with no `imageUrl`, an `imageUrl` outside the project's Cloudinary, or malformed body |

Maps to: `FR-CHAT-02`, `FR-CHAT-03`, `FR-CHAT-04`.

---

### PATCH /api/v1/conversations/{id}/read

Mark every unread message from the other participant in conversation `{id}` as read.

- Actor/Auth: participant only. Requires `Authorization: Bearer <access-token>`. No body.
- Idempotent: calling it again changes nothing.
- When at least one message changed, `MESSAGES_READ` is pushed to the other participant after commit.

Success: `200 OK` with `ApiResponse<null>`.

Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Authenticated actor is neither `CUSTOMER` nor `SELLER` |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Conversation does not exist or the caller is not a participant |

Maps to: `FR-CHAT-02`, `FR-CHAT-03`.
