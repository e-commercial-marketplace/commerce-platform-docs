# Cart API Contract

Scope: authenticated customer cart management before checkout. All responses use the `ApiResponse<T>` envelope and common error codes defined in [..\..\API_CONTRACT.md](..\..\API_CONTRACT.md) (§2 conventions, §5 cart).

Conventions: base path `/api/v1`; money as decimal string in `VND`. Cart items carry no price snapshot; checkout recalculates prices server-side. Rules: `(user, product)` is unique (POST on an existing product increments quantity), `quantity >= 1`, inactive products and insufficient stock are rejected.

## Endpoints (Confirmed)

### GET /api/v1/cart

Status: Confirmed (root §5, SRS §5.3).

- Actor/auth: `CUSTOMER` only, `Authorization: Bearer <access-token>` required. The server resolves the cart owner from the token; a customer sees only their own cart.
- Maps to: FR-CART-01.
- Success: `200 OK`, `ApiResponse<CartDto>`. `data` payload:

```json
{
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
      "sellerId": "seller-uuid"
    }
  ],
  "subtotal": "118.00"
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Authenticated actor is not `CUSTOMER` |

### POST /api/v1/cart/items

Status: Confirmed (root §5, SRS §5.3).

- Actor/auth: `CUSTOMER` only + own cart.
- Maps to: FR-CART-02, FR-CART-03, FR-CART-04, FR-CART-05.
- Business rule: if the product is already in the cart, the endpoint increments the existing row instead of creating a duplicate; the combined quantity is validated against current stock.
- Request body (JSON):

```json
{
  "productId": "product-uuid",
  "quantity": 1
}
```

- Success: `200 OK`, `ApiResponse<CartDto>` with the full updated cart (same shape as GET).
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `CUSTOMER` |
| `VALIDATION_ERROR` | 400 | Missing `productId` or `quantity < 1` |
| `RESOURCE_NOT_FOUND` | 404 | No product with `productId` |
| `PRODUCT_INACTIVE` | 400 | Product is unlisted/soft-deleted (`active = false`) |
| `INSUFFICIENT_STOCK` | 409 | Requested (combined) quantity exceeds `products.stock` |

### PATCH /api/v1/cart/items/{id}

Status: Confirmed (root §5, SRS §5.3).

- Actor/auth: `CUSTOMER` only + own cart row.
- Maps to: FR-CART-02, FR-CART-04.
- Params: `id` (path, cart-item UUID).
- Request body (JSON):

```json
{
  "quantity": 2
}
```

- Success: `200 OK`, `ApiResponse<CartDto>` with the full updated cart.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `CUSTOMER`, or cart item belongs to another customer |
| `VALIDATION_ERROR` | 400 | `quantity < 1` |
| `CART_ITEM_NOT_FOUND` | 404 | No cart item with `id` in the caller's cart |
| `PRODUCT_INACTIVE` | 400 | Referenced product became inactive |
| `INSUFFICIENT_STOCK` | 409 | Requested quantity exceeds current stock |

### DELETE /api/v1/cart/items/{id}

Status: Confirmed (root §5, SRS §5.3).

- Actor/auth: `CUSTOMER` only + own cart row.
- Maps to: FR-CART-02.
- Params: `id` (path, cart-item UUID). Empty request body.
- Success: `200 OK`, `ApiResponse<CartDto>` with the full updated cart.
- Errors:

| Code | HTTP | When |
|---|---|---|
| `AUTHENTICATION_REQUIRED` | 401 | Missing/expired/invalid token |
| `FORBIDDEN` | 403 | Not `CUSTOMER`, or cart item belongs to another customer |
| `CART_ITEM_NOT_FOUND` | 404 | No cart item with `id` in the caller's cart |
