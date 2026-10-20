# VNPAY Payment Integration — Sequence Diagram

- **Source:** [Payment API contract](../../features/payment/API_CONTRACT.md), predicted `POST /api/v1/payments/vnpay/create` and `POST /api/v1/payments/vnpay/callback`.
- **Scope:** supplemental integration flow; not one of the 60 use-case specifications.

> **Target sequence:** payment service/controller/repository classes are planned placeholders. VNPAY endpoints and payload details remain Predicted in the API contract; provider secrets stay on the backend.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as React Native client
    participant Security as Spring Security
    participant Controller as PaymentController
    participant Service as PaymentService
    participant OrderService as OrderService
    participant OrderRepo as OrderRepository
    participant PaymentRepo as PaymentRepository
    participant DB as PostgreSQL
    participant VNPAY as External VNPAY
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Choose online payment for an owned order
    Client->>Security: POST /api/v1/payments/vnpay/create + customer JWT
    Security->>Controller: Forward authenticated CUSTOMER principal
    Controller->>Service: Create a VNPAY payment attempt
    Service->>OrderService: Verify ownership and payable order state
    OrderService->>OrderRepo: Load caller-owned order
    OrderRepo->>DB: SELECT order and payment state
    DB-->>OrderRepo: Order state
    OrderRepo-->>OrderService: Owned payable order
    OrderService-->>Service: Order payable
    Service->>PaymentRepo: Persist a new PENDING attempt and unique transaction reference
    PaymentRepo->>DB: INSERT payment attempt
    DB-->>PaymentRepo: Persisted attempt
    Service->>VNPAY: Send signed payment request (secret remains server-side)
    VNPAY-->>Service: Return redirect URL/parameters
    Service-->>Controller: Result DTO
    Controller-->>Client: 200 OK ApiResponse<VnpayCreateResponse>
    Client->>VNPAY: Redirect customer to payment URL
    VNPAY->>Controller: POST /api/v1/payments/vnpay/callback
    Controller->>Service: Process callback
    Service->>Service: Validate signature, amount, transaction idempotency
    alt Valid, new successful callback
        Service->>PaymentRepo: Mark attempt COMPLETED
        PaymentRepo->>DB: UPDATE payment and coordinate order state transactionally
        DB-->>PaymentRepo: Updated payment/order state
        PaymentRepo-->>Service: Completed state
        Service-->>Controller: Result
        Controller-->>VNPAY: 200 OK ApiResponse<PaymentResponse>
    else Invalid signature, mismatch, or duplicate callback
        Service-->>Controller: Callback result/error
        Controller->>Handler: Normalize callback validation error when rejected
        Handler-->>VNPAY: Contract-defined 4xx/200 response
    end
    VNPAY-->>Client: Redirect to the configured return URL (status source remains backend callback)
    Client-->>Customer: Display current payment status
```
