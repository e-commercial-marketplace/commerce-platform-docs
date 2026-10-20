# Review API Contract

## 1. Scope

This contract covers the `review` feature boundary: eligibility, creation, and public listing of
product reviews.

Planned surface is exactly 2 endpoints:

```text
POST /api/v1/products/{productId}/reviews
GET  /api/v1/products/{productId}/reviews
```

MVP boundary: one eligible customer review containing `rating` and `comment` only. Review tags and
image attachments are deferred.

Hard rule: no `DELETE` review endpoint exists for any role. In particular, `ADMIN` has no MVP
action on reviews. The `Permissions` table in the feature `README.md` mentions future moderation
only as a workflow placeholder; it does not define an API operation.

Common wire envelope (`ApiResponse<T>`), pagination shape (`PageResponse<T>`), data formats (UUID,
ISO-8601 UTC, zero-based `page`), and shared error codes/HTTP mapping are defined in
[`..\..\API_CONTRACT.md`](../../API_CONTRACT.md) (§2) and are not repeated here except for
endpoint-specific behavior.

Feature source: `features/review/README.md`. No field outside that README is introduced below.

---

### POST /api/v1/products/{productId}/reviews

Create one review for a product.

- Actor/Auth: `CUSTOMER` only. Requires `Authorization: Bearer <access-token>`.
- Eligibility (all required):
  - caller purchased the product (`FR-REV-01`);
  - the purchased product was received/delivered;
  - no existing review for unique `(user, product)` (`FR-REV-02`).
- The review author (`userId`) is resolved from the authentication principal. The client must not
  supply `userId`.

Path parameters:

| Parameter | Type | Meaning |
|---|---|---|
| `productId` | UUID string | Reviewed product |

Request body (raw JSON):

```json
{
  "rating": 5,
  "comment": "Good product and fast delivery"
}
```

| Field | Type | Rule |
|---|---|---|
| `rating` | integer | Required, `1..5` (`FR-REV-03`) |
| `comment` | string | Customer feedback text |

Success: `201 Created` with `ApiResponse<ReviewResponse>`. The `data` object is:

```json
{
  "id": "review-uuid",
  "productId": "product-uuid",
  "userId": "customer-uuid",
  "rating": 5,
  "comment": "Good product and fast delivery",
  "createdAt": "2026-10-14T10:30:00Z"
}
```

Errors (common envelope in [`..\..\API_CONTRACT.md`](../../API_CONTRACT.md)):

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | `401 Unauthorized` | Missing, expired, or invalid token |
| `FORBIDDEN` | `403 Forbidden` | Authenticated actor is not `CUSTOMER` |
| `REVIEW_NOT_ELIGIBLE` | `403 Forbidden` | Customer did not purchase and receive the product |
| `REVIEW_ALREADY_EXISTS` | `409 Conflict` | Unique `(user, product)` already has a review |
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Product does not exist or is not visible |
| `VALIDATION_ERROR` | `400 Bad Request` | `rating` outside `1..5` or malformed body |

Maps to: `FR-REV-01`, `FR-REV-02`, `FR-REV-03`.

---

### GET /api/v1/products/{productId}/reviews

List public reviews for a product.

- Actor/Auth: public. No token required.
- Only public review fields are exposed (same `ReviewResponse` shape as creation).

Query parameters (all optional):

| Parameter | Meaning |
|---|---|
| `page` | Zero-based page index; default `0` |
| `size` | Page size; default `20`, backend applies a maximum |

Success: `200 OK` with `ApiResponse<PageResponse<ReviewResponse>>`:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": "review-uuid",
        "productId": "product-uuid",
        "userId": "customer-uuid",
        "rating": 5,
        "comment": "Good product and fast delivery",
        "createdAt": "2026-10-14T10:30:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1
  },
  "error": null,
  "timestamp": "2026-10-14T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Errors:

| Code | HTTP | When |
|---|---|---|
| `RESOURCE_NOT_FOUND` | `404 Not Found` | Product does not exist or is not visible |
| `VALIDATION_ERROR` | `400 Bad Request` | Invalid `page`/`size` |

Maps to: `FR-REV-02` (read path of the unique review model), `FR-REV-03` (rating range as stored).
