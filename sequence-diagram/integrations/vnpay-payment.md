# VNPAY Payment Integration — Sequence Diagram

- **Source:** [Payment API contract](../../features/payment/API_CONTRACT.md) and [Order API contract](../../features/order/API_CONTRACT.md): `POST /api/v1/orders` (VNPAY), `POST /api/v1/payments/vnpay/create`, `GET /api/v1/payments/vnpay/ipn`, `GET /api/v1/payments/vnpay/return` — Confirmed.
- **Scope:** supplemental integration flow covering checkout with VNPAY, both result channels, and the 24-hour expiry job; the customer use case is [customer/pay-for-order-vnpay.md](../customer/pay-for-order-vnpay.md).

> The hash secret stays on the backend. The IPN only reaches a public deployment (registered in the VNPAY merchant portal); in local development the Return URL records the result through the same handler.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as React Native client
    participant Backend as Backend (controllers + services)
    participant DB as PostgreSQL
    participant VNPAY as VNPAY sandbox
    participant Job as PrepaymentExpiryJob
    Customer->>Client: Check out selected cart lines with VNPAY
    Client->>Backend: POST /api/v1/orders {cartItemIds, shippingAddress, paymentMethod: VNPAY}
    Backend->>DB: Decrease stock, INSERT order and items, INSERT payment (VNPAY, PENDING, vnp_TxnRef)
    Backend->>Backend: Sign vnp_* parameters (HMAC-SHA512, link valid 15 minutes, never past the 24-hour deadline)
    Backend-->>Client: 201 OrderResponse with paymentUrl
    Client->>VNPAY: Open paymentUrl in the browser
    Customer->>VNPAY: Pay with the test card
    par Server-to-server
        VNPAY->>Backend: GET /api/v1/payments/vnpay/ipn?vnp_...
        Backend->>DB: Verify, lock order, record result once
        Backend-->>VNPAY: 200 {"RspCode":"00","Message":"Confirm Success"} (02 if already recorded)
    and Browser redirect
        VNPAY->>Backend: GET /api/v1/payments/vnpay/return?vnp_...
        Backend->>DB: Same handler; records the result if the IPN has not yet
        Backend-->>Client: 302 to app-redirect-url?orderId=...&status=...
    end
    Client->>Backend: GET /api/v1/orders/{id}
    Backend-->>Client: status PAID, paymentStatus COMPLETED (or FAILED, payable again)
    Client-->>Customer: Display the result
    loop Every 5 minutes
        Job->>DB: Find PENDING VNPAY orders older than 24 hours without a COMPLETED payment
        Job->>Backend: Cancel each order in its own transaction
        Backend->>DB: Lock order, cancel remaining lines, restore stock once, pending payments FAILED, order CANCELLED
    end
```
