# Upload - Upload Image — Sequence Diagram

- **Use-case spec:** [upload-image.vi.md](../../usecase-specification/customer/upload-image.vi.md)
- **API route/status:** `POST /api/v1/uploads/signature, then POST to Cloudinary` — Confirmed
- **Feature API contract:** [features/upload/API_CONTRACT.md](../../features/upload/API_CONTRACT.md)

> The file never passes through the backend. The returned URL is then sent as imageUrl to the product or chat endpoint.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as UploadController
    participant Service as UploadService
    participant Users as UserRepository
    participant Gateway as CloudinaryGateway
    participant Cloud as Cloudinary
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Choose an image
    Client->>Security: POST /api/v1/uploads/signature with purpose CHAT
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Token accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: createSignature(caller, purpose)
        alt Allowed and configured
            Service->>Service: Check the caller is CUSTOMER or SELLER
            Service->>Gateway: sign(publicId, timestamp)
            Gateway-->>Service: SHA-1 signature of the sorted parameters plus the secret
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<UploadSignatureResponse>
            Client->>Cloud: POST the file with the signed parameters
            Cloud-->>Client: secure_url
            Client-->>Customer: Display the image
        else Caller must be CUSTOMER or SELLER
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 403 FORBIDDEN
            Client-->>Customer: Display the error
        else Cloudinary is not configured
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 503 UPLOAD_NOT_CONFIGURED
            Client-->>Customer: Display the error
        end
    end
```
