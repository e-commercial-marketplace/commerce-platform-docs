# Order - Checkout Order — Sequence Diagram

- **Use-case spec:** [checkout-order.vi.md](../../usecase-specification/customer/checkout-order.vi.md)
- **API route/status:** `POST /api/v1/orders` — Confirmed
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Client application
    participant Security as Spring Security
    participant Controller as OrderController
    participant Service as OrderService
    participant Users as UserRepository
    participant Cart as CartItemRepository
    participant Products as ProductService
    participant Vouchers as VoucherService
    participant Repo as OrderRepository
    participant Payments as PaymentService
    participant VNPAY as VnpayGateway
    participant Cache as Redis cache
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Checkout Order
    Client->>Security: POST /api/v1/orders
    alt Token missing, expired or invalid
        Security-->>Client: 401 AUTHENTICATION_REQUIRED
        Client-->>Customer: Display the error
    else Caller is not CUSTOMER
        Security-->>Client: 403 FORBIDDEN
        Client-->>Customer: Display the error
    else Token and role accepted
        Security->>Controller: Forward the authenticated CUSTOMER principal
        Controller->>Service: checkout(callerId, request, clientIp)
        alt Request succeeds
            Service->>Users: Lock the customer row
            Users->>DB: SELECT users FOR UPDATE
            Service->>Cart: Load the chosen lines of this customer
            Cart->>DB: SELECT cart_items
            loop Each line, ordered by product id
                Service->>Products: requireListed and decreaseStock
                Products->>DB: Conditional UPDATE products SET stock
            end
            opt voucherCode was sent
                Service->>Vouchers: useVoucher(code)
                Vouchers->>DB: Conditional UPDATE vouchers SET times_used
            end
            Service->>Repo: Save the order with snapshot lines and total
            Repo->>DB: INSERT orders and order_items
            alt paymentMethod is VNPAY
                Service->>Payments: createVnpayPayment
                Payments->>DB: INSERT payments
                Payments->>VNPAY: Build the signed payment URL
            else paymentMethod is COD
                Service->>Payments: createCodPayment
                Payments->>DB: INSERT payments
            end
            Service->>Cart: Delete the bought lines
            Cart->>DB: DELETE cart_items
            Service->>Cache: Evict the cart after commit
            Service-->>Controller: Use-case result
            Controller-->>Client: 201 Created ApiResponse<OrderResponse> (paymentUrl for VNPAY)
            Client-->>Customer: Display the result
        else Invalid body or duplicate ids
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 VALIDATION_ERROR
            Client-->>Customer: Display the error
        else A line is unknown or foreign
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 CART_ITEM_NOT_FOUND
            Client-->>Customer: Display the error
        else A product is hidden
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 400 PRODUCT_INACTIVE
            Client-->>Customer: Display the error
        else Not enough stock, whole transaction rolls back
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 INSUFFICIENT_STOCK
            Client-->>Customer: Display the error
        else Unknown voucher
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 404 RESOURCE_NOT_FOUND
            Client-->>Customer: Display the error
        else Voucher out of validity or used up
            Service-->>Controller: Propagate the error
            Controller->>Handler: Normalize safe API error
            Handler-->>Client: 409 VOUCHER_EXPIRED or VOUCHER_EXHAUSTED
            Client-->>Customer: Display the error
        end
    end
```
