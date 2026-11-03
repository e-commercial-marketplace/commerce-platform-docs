# Voucher - View Vouchers — Sequence Diagram

- **Use-case spec:** [view-vouchers.vi.md](../../usecase-specification/customer/view-vouchers.vi.md)
- **API route/status:** `GET /api/v1/vouchers` — Confirmed
- **Feature API contract:** [features/voucher/API_CONTRACT.md](../../features/voucher/API_CONTRACT.md)

> Public endpoint. Copying a code happens only on the device. Using a voucher still requires a signed-in Customer at checkout.

```mermaid
sequenceDiagram
    autonumber
    actor User as Guest or Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as VoucherController
    participant Service as VoucherService
    participant Repo as VoucherRepository
    participant DB as PostgreSQL
    User->>Client: Open the home screen or view all vouchers
    Client->>Security: GET /api/v1/vouchers?page&size
    Security->>Controller: Forward the request (public GET)
    Controller->>Service: listUsableVouchers(page, size)
    alt Request succeeds
        Service->>Repo: findUsable(now, pageable)
        Repo->>DB: SELECT vouchers within validity with uses left, highest percent first
        Service-->>Controller: Use-case result
        Controller-->>Client: 200 OK ApiResponse<PageResponse<VoucherSummaryResponse>>
        Client-->>User: Display the vouchers
        User->>Client: Copy a code
        Client-->>User: Confirm the code is copied
    else Invalid page or size
        Controller-->>Client: 400 VALIDATION_ERROR
        Client-->>User: Display the error
    end
```
