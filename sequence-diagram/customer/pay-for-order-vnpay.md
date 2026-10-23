# Payment - Pay for Order (VNPAY) — Sequence Diagram

- **Use-case spec:** [pay-for-order-vnpay.vi.md](../../usecase-specification/customer/pay-for-order-vnpay.vi.md)
- **API route/status:** `POST /api/v1/payments/vnpay/create`, `GET /api/v1/payments/vnpay/ipn`, `GET /api/v1/payments/vnpay/return` — Confirmed
- **Feature API contract:** [features/payment/API_CONTRACT.md](../../features/payment/API_CONTRACT.md)

> **Scope:** this diagram shows paying again from the order detail (retry). The first attempt is created by checkout; the full checkout-to-result flow, including the 24-hour expiry job, is in [integrations/vnpay-payment.md](../integrations/vnpay-payment.md). IPN and Return URL run the same handler, so only one result path is drawn.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as PaymentController
    participant Service as PaymentService
    participant Repo as OrderRepository / PaymentRepository
    participant DB as PostgreSQL
    participant VNPAY as VNPAY sandbox
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Pay the order again with VNPAY
    Client->>Security: POST /api/v1/payments/vnpay/create with orderId and customer JWT
    Security->>Controller: Forward authenticated CUSTOMER principal
    Controller->>Service: Create a VNPAY retry with orderId, caller id and client IP
    Service->>Repo: Load the caller-owned order and lock it
    Repo->>DB: SELECT order by id and customer, then SELECT ... FOR UPDATE
    DB-->>Repo: Order with its payments
    Repo-->>Service: Locked order
    alt Order PENDING, paid by VNPAY, within 24 hours
        Service->>Repo: Mark pending VNPAY attempts FAILED, insert a new PENDING attempt with a new vnp_TxnRef
        Repo->>DB: UPDATE payments, INSERT payments
        DB-->>Repo: Persisted attempt
        Service->>Service: Build the vnp_* parameters and sign them with HMAC-SHA512 (secret stays on the server)
        Service-->>Controller: Payment link
        Controller-->>Client: 200 OK ApiResponse<VnpayPaymentResponse>
        Client->>VNPAY: Open paymentUrl
        Customer->>VNPAY: Enter the test card and OTP
        VNPAY->>Controller: GET /api/v1/payments/vnpay/ipn (or /return) with signed vnp_* fields
        Controller->>Service: Handle the VNPAY result
        Service->>Service: Verify the signature
        Service->>Repo: Find the attempt by vnp_TxnRef and lock its order
        Repo->>DB: SELECT payment, SELECT order FOR UPDATE
        DB-->>Repo: Attempt and order
        Repo-->>Service: Attempt and locked order
        alt Signature, reference and amount valid, attempt still PENDING
            Service->>Repo: Success code: attempt COMPLETED and order PAID; failure code: attempt FAILED
            Repo->>DB: UPDATE payments (and orders)
            DB-->>Repo: Updated rows
            Service-->>Controller: RspCode 00
        else Already processed
            Service-->>Controller: RspCode 02, nothing changed
        else Invalid signature, unknown reference, or wrong amount
            Service-->>Controller: RspCode 97, 01 or 04, nothing changed
        end
        Controller-->>VNPAY: IPN: 200 {"RspCode","Message"}
        Controller-->>Client: Return: 302 to the app URL with orderId and status
        Client->>Security: GET /api/v1/orders/{id}
        Client-->>Customer: Display the payment result
    else Order not payable
        Service-->>Controller: Propagate ORDER_NOT_PAYABLE
        Controller->>Handler: Normalize safe API error
        Handler-->>Client: 409 ORDER_NOT_PAYABLE
        Client-->>Customer: Display the error
    end
```
