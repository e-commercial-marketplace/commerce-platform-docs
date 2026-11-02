# Image Upload Feature

## Purpose

Let sellers and chat participants attach real images without passing files through the backend.

## Responsibilities

- Issue a short-lived Cloudinary upload signature to callers allowed to upload.
- Reject product and chat image URLs that do not belong to the project's Cloudinary account.

## API

```text
POST /api/v1/uploads/signature
```

Details: [`API_CONTRACT.md`](API_CONTRACT.md).

## Important rules

- The API secret stays on the backend; the client only receives the signature.
- The backend chooses `public_id`, so clients cannot overwrite existing images.
- No table is added: products and messages keep storing `image_url`.
- Empty `CLOUDINARY_*` settings turn the feature off: the signature endpoint answers `503` and URLs are not checked.
