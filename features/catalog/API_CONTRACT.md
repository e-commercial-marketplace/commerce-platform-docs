# Catalog API Contract

Scope: public product/category browsing plus seller product management and admin category management. All responses use the `ApiResponse<T>` envelope and common error codes defined in [..\..\API_CONTRACT.md](..\..\API_CONTRACT.md) (§2 conventions, §4 catalog).

Conventions: base path `/api/v1`; money as decimal string in `VND`; `price > 0`, `stock >= 0`; `active = false` means soft-deleted/unlisted and hidden from public reads. Product fields follow the catalog feature model: `products{id, name, description, imageUrl, price, stock, ratingAverage, reviewCount, active, categoryId, sellerId}`. MVP supports one `imageUrl` and no variants.

## Public catalog (Confirmed)

### GET /api/v1/products

Status: Confirmed (root §4, SRS §5.2).

- Actor/auth: public, no token required.
- Maps to: FR-CAT-01, FR-CAT-02.
- Query params (all optional):

| Parameter | Meaning |
|---|---|
| `q` | Text search against product name and description |
| `categoryId` | Category filter (UUID) |
| `minPrice` / `maxPrice` | Inclusive price range |
| `minRating` | Minimum rating `0`–`5` |
| `sort` | `relevance`, `price_asc`, `price_desc`, or `rating_desc` |
| `page` | Zero-based page index, default `0` |
| `size` | Page size, default `20` |

- Default sort is `relevance` when `q` is present, `rating_desc` otherwise; deterministic `id ASC` tie-breaker.
- Success: `200 OK`, `ApiResponse<PageResponse<ProductSummaryResponse>>`. `data` payload:

```json
{
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
      "category": { "id": "category-uuid", "name": "Electronics" },
      "sellerId": "seller-uuid"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Invalid `sort`, `minRating`, price range, or pagination |

### GET /api/v1/products/{id}

Status: Confirmed (root §4, SRS §5.2).

- Actor/auth: public, no token required.
- Maps to: FR-CAT-03.
- Params: `id` (path, product UUID).
- Success: `200 OK`, `ApiResponse<ProductResponse>` — same product fields as the list item above.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `RESOURCE_NOT_FOUND` | 404 | No active product with `id` |

### GET /api/v1/categories

Status: Confirmed (root §4, SRS §5.2).

- Actor/auth: public, no token required.
- Maps to: FR-CAT-03.
- Success: `200 OK`, `ApiResponse<List<CategorySummaryResponse>>`. `data` payload:

```json
[
  { "id": "category-uuid", "name": "Electronics" }
]
```

- Errors: none specific besides `INTERNAL_ERROR` (500).

## Seller products (Confirmed)

All endpoints below require `Authorization: Bearer <access-token>` with role `SELLER` and `sellerStatus = APPROVED`. Ownership rule: `product.sellerId == authenticatedSeller.id` (FR-CAT-06).

### POST /api/v1/seller/products

Status: Confirmed (SRS §5.2).

- Actor/auth: `SELLER` (`APPROVED`) only.
- Maps to: FR-CAT-04, FR-CAT-05.
- Request body (JSON):

```json
{
  "name": "Headphone",
  "description": "Wireless headphone",
  "imageUrl": "https://example.com/headphone.jpg",
  "price": "59.00",
  "stock": 12,
  "categoryId": "category-uuid",
  "active": true
}
```

- Success: `201 Created`, `ApiResponse<ProductResponse>` with the created product (including `id`, `ratingAverage: "0"`, `reviewCount: 0`, `sellerId` from token).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not a seller, or seller not `APPROVED` |
| `VALIDATION_ERROR` | 400 | `price <= 0`, `stock < 0`, missing name/category, unknown `categoryId` |

### PATCH /api/v1/seller/products/{id}

Status: Confirmed (SRS §5.2).

- Actor/auth: `SELLER` (`APPROVED`) + own product only.
- Maps to: FR-CAT-04, FR-CAT-06.
- Request body (JSON, all fields optional):

```json
{
  "name": "Headphone Pro",
  "description": "Updated description",
  "imageUrl": "https://example.com/headphone-pro.jpg",
  "price": "69.00",
  "categoryId": "category-uuid",
  "active": true
}
```

- Success: `200 OK`, `ApiResponse<ProductResponse>` with the updated product. Use the stock endpoint for stock changes.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not an approved seller, or product owned by another seller |
| `RESOURCE_NOT_FOUND` | 404 | No product with `id` |
| `VALIDATION_ERROR` | 400 | Invalid field values or unknown `categoryId` |

### DELETE /api/v1/seller/products/{id}

Status: Confirmed (SRS §5.2).

- Actor/auth: `SELLER` (`APPROVED`) + own product only.
- Maps to: FR-CAT-04, FR-CAT-06.
- Business rule: soft-delete/unlist (`active = false`); historical order and review references are kept.
- Success: `204 No Content` (no body).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not an approved seller, or product owned by another seller |
| `RESOURCE_NOT_FOUND` | 404 | No product with `id` |

### PATCH /api/v1/seller/products/{id}/stock

Status: Confirmed (SRS §5.2).

- Actor/auth: `SELLER` (`APPROVED`) + own product only.
- Maps to: FR-CAT-04, FR-CAT-06.
- Request body (JSON):

```json
{
  "stock": 25
}
```

- Success: `200 OK`, `ApiResponse<ProductResponse>` with the updated `stock`.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not an approved seller, or product owned by another seller |
| `RESOURCE_NOT_FOUND` | 404 | No product with `id` |
| `VALIDATION_ERROR` | 400 | `stock < 0` or missing |

## Admin categories (Predicted)

All endpoints below require `ADMIN`. Category fields: `categories{id, name}` (`name` unique).

### POST /api/v1/admin/categories

Status: Predicted.

- Actor/auth: `ADMIN` only.
- Maps to: FR-CAT-05 (category ownership invariant).
- Request body (JSON):

```json
{
  "name": "Electronics"
}
```

- Success: `201 Created`, `ApiResponse<CategorySummaryResponse>`:

```json
{
  "id": "category-uuid",
  "name": "Electronics"
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `VALIDATION_ERROR` | 400 | Missing/blank or duplicate `name` |

### PATCH /api/v1/admin/categories/{id}

Status: Predicted.

- Actor/auth: `ADMIN` only.
- Maps to: FR-CAT-05.
- Request body (JSON):

```json
{
  "name": "Consumer Electronics"
}
```

- Success: `200 OK`, `ApiResponse<CategorySummaryResponse>` with the renamed category.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `RESOURCE_NOT_FOUND` | 404 | No category with `id` |
| `VALIDATION_ERROR` | 400 | Missing/blank or duplicate `name` |

### DELETE /api/v1/admin/categories/{id}

Status: Predicted.

- Actor/auth: `ADMIN` only.
- Maps to: FR-CAT-05.
- Business rule: a category that still contains products must not be deleted (reject or require reassignment per implementation).
- Success: `204 No Content` (no body).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `ADMIN` |
| `RESOURCE_NOT_FOUND` | 404 | No category with `id` |
| `VALIDATION_ERROR` | 400 | Category still has products |
