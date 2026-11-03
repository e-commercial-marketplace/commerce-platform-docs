# Voucher - Apply Voucher — Sequence Diagram

- **Use-case spec:** [apply-voucher.vi.md](../../usecase-specification/customer/apply-voucher.vi.md)
- **API route/status:** `GET /api/v1/vouchers/{code}/validate` — Confirmed
- **Feature API contract:** [features/voucher/API_CONTRACT.md](../../features/voucher/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as VoucherController
    participant Service as VoucherService
    participant Repo as VoucherRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Choose a suggested voucher or enter a code
    Client->>Security: GET /api/v1/vouchers/{code}/validate
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: checkVoucherUsable(code)
        alt Request succeeds
            Service->>Repo: findById(normalized code)
            Repo->>DB: SELECT vouchers
            Service->>Service: Check validFrom, validTo and timesUsed
            Service-->>Controller: Use-case result
            Controller-->>Client: 200 OK ApiResponse<VoucherResponse>
            Client-->>Customer: Display the result
        else Unknown code
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Customer: Display the error
        else Not started or expired
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 VOUCHER_EXPIRED
            Client-->>Customer: Display the error
        else No use left
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 VOUCHER_EXHAUSTED
            Client-->>Customer: Display the error
        end
    end
```
