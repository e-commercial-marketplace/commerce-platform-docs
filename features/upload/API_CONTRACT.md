# Image Upload API Contract

Status: Confirmed (backend implemented). The mobile client still has to adopt this flow.

Conventions: base path `/api/v1`; responses use `ApiResponse<T>`. Files never pass through the backend: the
client uploads directly to Cloudinary with a signature issued here, then sends the returned URL to the normal
product or chat endpoint.

## Flow

1. The client asks for a signature: `POST /api/v1/uploads/signature`.
2. The client posts the file to `uploadUrl` as `multipart/form-data` with `file`, `api_key`, `timestamp`,
   `public_id`, `allowed_formats`, `overwrite` and `signature`, using exactly the returned values.
3. Cloudinary answers with `secure_url`.
4. The client sends `secure_url` as `imageUrl` to the product or chat endpoint.

## POST /api/v1/uploads/signature

- Actor/auth: authenticated user, depending on `purpose`.
- Business rule: the backend generates `publicId` (a random UUID under `marketplace/products/` or
  `marketplace/chat/`), so a client cannot choose it or overwrite an existing image. The API secret never leaves
  the backend. A signature is valid for one hour from `timestamp`.
- Request body (JSON):

```json
{ "purpose": "PRODUCT" }
```

| `purpose` | Who may call |
|---|---|
| `PRODUCT` | `SELLER` with status `APPROVED` (read from the database) |
| `CHAT` | `CUSTOMER` or `SELLER` |

- Success: `200 OK`, `ApiResponse<UploadSignatureResponse>`. `data` payload:

```json
{
  "uploadUrl": "https://api.cloudinary.com/v1_1/demo/image/upload",
  "apiKey": "123456789012345",
  "timestamp": 1793600000,
  "publicId": "marketplace/products/6f1c2d3e-0000-4000-8000-000000000000",
  "allowedFormats": "jpg,png,webp",
  "overwrite": false,
  "signature": "bfd09f95f331f558cbd1320e67aa8d488770583e"
}
```

- Errors:

| Code | HTTP | When |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing or unknown `purpose` |
| `AUTHENTICATION_REQUIRED` | 401 | Missing, expired or invalid token |
| `FORBIDDEN` | 403 | The caller may not upload for this `purpose` |
| `UPLOAD_NOT_CONFIGURED` | 503 | `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY` or `CLOUDINARY_API_SECRET` is not set |

## Image URL rule

When Cloudinary is configured, a product or chat `imageUrl` must be
`https://res.cloudinary.com/<cloud name>/image/upload/...`, otherwise the endpoint answers
`400 VALIDATION_ERROR`.

| Endpoint | Checked when |
|---|---|
| `POST /conversations/{id}/messages` | `imageUrl` is sent |
| `POST /seller/products` | `imageUrl` is sent |
| `PATCH /seller/products/{id}` | `imageUrl` differs from the stored value, so older products keep working |

When Cloudinary is not configured, the URL is not checked (local development and tests).

## Known limits

- Cloudinary has no signed parameter for the maximum file size: the account limit applies, and the client should
  compress images first.
- Signature requests are not rate limited.
- An image that was uploaded but never used stays in Cloudinary.
