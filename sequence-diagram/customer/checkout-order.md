# Order - Checkout Order — Sequence Diagram

- **Use-case spec:** [checkout-order.vi.md](../../usecase-specification/customer/checkout-order.vi.md)
- **API route/status:** `POST /api/v1/orders` — Confirmed
- **Feature API contract:** [features/order/API_CONTRACT.md](../../features/order/API_CONTRACT.md)

> **Target sequence:** controller/service/repository names are planned placeholders; the current backend scaffold does not yet implement these layers.

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant Client as Customer client
    participant Security as Spring Security
    participant Controller as OrderController
    participant Service as OrderService
    participant CartService as CartService
    participant CartRepo as CartItemRepository
    participant CatalogService as CatalogService
    participant ProductRepo as ProductRepository
    participant OrderRepo as OrderRepository
    participant ItemRepo as OrderItemRepository
    participant VoucherService as VoucherService
    participant VoucherRepo as VoucherRepository
    participant DB as PostgreSQL
    participant Handler as GlobalExceptionHandler
    Customer->>Client: Submit shipping address and COD checkout
    Client->>Security: POST /api/v1/orders + customer JWT
    Security->>Controller: Forward authenticated CUSTOMER principal
    Controller->>Service: createOrder(request, principal)
    alt Cart, products, address, and optional voucher are valid
        Service->>CartService: Load the caller-owned cart
        CartService->>CartRepo: Find cart items for authenticated customer
        CartRepo->>DB: SELECT cart rows
        DB-->>CartRepo: Cart items
        CartRepo-->>CartService: Cart items
        CartService-->>Service: Cart snapshot source
        Service->>CatalogService: Validate active products, prices, and stock
        CatalogService->>ProductRepo: Read products and reserve/decrease stock
        ProductRepo->>DB: SELECT/UPDATE products in checkout transaction
        DB-->>ProductRepo: Current product state and updated stock
        ProductRepo-->>CatalogService: Validated product data
        CatalogService-->>Service: Validated line data
        opt voucherCode is present
            Service->>VoucherService: Validate and consume voucher transactionally
            VoucherService->>VoucherRepo: Validate usage and update consumption
            VoucherRepo->>DB: SELECT/UPDATE voucher within transaction
            DB-->>VoucherRepo: Voucher state
            VoucherRepo-->>VoucherService: Discount result
            VoucherService-->>Service: Discount amount
        end
        Service->>OrderRepo: Persist order totals and address snapshot
        OrderRepo->>DB: INSERT orders
        Service->>ItemRepo: Persist purchased item snapshots
        ItemRepo->>DB: INSERT order_items
        Service->>CartService: Clear the purchased cart rows in the same transaction
        CartService->>CartRepo: Delete purchased cart items for the authenticated customer
        CartRepo->>DB: DELETE purchased cart_items
        DB-->>CartRepo: Deleted rows
        CartRepo-->>CartService: Cart cleared
        CartService-->>Service: Cart cleared
        Note over Service,DB: All reads and writes above join the single checkout transaction, which commits after the cart is cleared.
        Service-->>Controller: Created order result
        Controller-->>Client: 201 Created ApiResponse<OrderResponse>
        Client-->>Customer: Display the created order
    else Invalid address/cart/product/stock/voucher
        Service-->>Controller: Propagate exception, transaction rolls back
        Controller->>Handler: Map exception to safe API error
        Handler-->>Client: Error ApiResponse, no partial checkout writes
        Client-->>Customer: Show checkout error
    end
```
