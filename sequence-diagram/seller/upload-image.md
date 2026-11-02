# Upload - Upload Image — Sequence Diagram

- **Use-case spec:** [upload-image.vi.md](../../usecase-specification/seller/upload-image.vi.md)
- **API route/status:** `POST /api/v1/uploads/signature, then POST to Cloudinary` — Confirmed
- **Feature API contract:** [features/upload/API_CONTRACT.md](../../features/upload/API_CONTRACT.md)

> The file never passes through the backend. The returned URL is then sent as imageUrl to the product or chat endpoint.

```mermaid
sequenceDiagram
    autonumber
    actor Seller
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as UploadController
    participant Service as UploadService
    participant Users as UserRepository
    participant Gateway as CloudinaryGateway
    participant Cloud as Cloudinary
    participant Handler as GlobalExceptionHandler
    Seller->>Client: Choose an image
    Client->>Security: POST /api/v1/uploads/signature with purpose PRODUCT
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Seller: Display the error
    else Token accepted
        Security->>Controller: Forward the authenticated SELLER principal
        Controller->>Service: createSignature(caller, purpose)
        alt Allowed and configured
            Service->>Users: getApprovedSeller(token id)
            Service->>Gateway: sign(publicId, timestamp)
            Gateway-->>Service: SHA-1 signature of the sorted parameters plus the secret
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<UploadSignatureResponse>
            Client->>Cloud: POST the file with the signed parameters
            Cloud-->>Client: secure_url
            Client-->>Seller: Display the image
        else Seller must be APPROVED
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Seller: Display the error
        else Cloudinary is not configured
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 503 UPLOAD_NOT_CONFIGURED
            Client-->>Seller: Display the error
        end
    end
```
