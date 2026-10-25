# Sequence Diagrams

This folder contains one technical Mermaid sequence diagram per use case in
[`usecase-specification/`](../usecase-specification/) (64 use cases), plus one supplemental VNPAY
integration diagram. Unlike the activity diagrams, these diagrams are not only business flows:
they trace the planned request through the real backend layers.

## Target request path

```text
Client -> Spring Security -> FeatureController -> FeatureService -> FeatureRepository -> PostgreSQL
```

- `FeatureService` is the feature's service layer: it receives the controller call, applies the
  business rules, and reads or writes through the feature's repository. Each diagram shows it as a
  single participant; the Spring interface/implementation split is an internal detail and is not
  drawn as a separate hop.
- Cross-feature work calls the owning feature's service (for example the order service calling
  `CartService`); a feature never reaches into another feature's repository or entities.
- External providers (`VNPAY`, LLM provider, email) appear only in the diagrams whose contract or
  spec requires them, and provider secrets stay on the backend.
- `GlobalExceptionHandler` is the shared error path; it returns the contract-defined
  `ApiResponse<T>` error and must not leak internal detail.

> **Target design:** the current backend scaffold does not yet implement the controller/service/
> repository layers. Every diagram states its endpoint status (Confirmed / Predicted / TBD / Phased)
> next to the route. When no contract exists, the diagram stops instead of inventing a route or an
> external call.

## Files

### Guest

| Diagram | Use case | API route (status) |
|---|---|---|
| [`guest/browse-public-catalog.md`](guest/browse-public-catalog.md) | Catalog - Browse Public Catalog | `GET /api/v1/products and GET /api/v1/categories` (Confirmed) |
| [`guest/login.md`](guest/login.md) | Auth - Guest Login | `POST /api/v1/auth/login` (Confirmed) |
| [`guest/login-with-google.md`](guest/login-with-google.md) | Auth - Guest Login with Google | `POST /api/v1/auth/google` (Planned) |
| [`guest/register-customer-account.md`](guest/register-customer-account.md) | Auth - Register Customer Account | `POST /api/v1/auth/register` (Confirmed) |
| [`guest/search-and-filter-products.md`](guest/search-and-filter-products.md) | Catalog - Search and Filter Products | `GET /api/v1/products (query/filter parameters per catalog contract)` (Confirmed) |
| [`guest/use-ai-product-search.md`](guest/use-ai-product-search.md) | AI - Search Products with AI Assistant | `POST /api/v1/ai/ask` (Planned) |
| [`guest/view-product-details.md`](guest/view-product-details.md) | Catalog - View Product Details | `GET /api/v1/products/{id}` (Confirmed) |

### Customer

| Diagram | Use case | API route (status) |
|---|---|---|
| [`customer/add-cart-item.md`](customer/add-cart-item.md) | Cart - Add Cart Item | `POST /api/v1/cart/items` (Confirmed) |
| [`customer/browse-catalog.md`](customer/browse-catalog.md) | Catalog - Browse Catalog | `GET /api/v1/products and GET /api/v1/categories` (Confirmed) |
| [`customer/chat-with-seller.md`](customer/chat-with-seller.md) | Chat - Chat with Seller | `WebSocket /ws/chat (message frames are phased)` (Phased) |
| [`customer/checkout-order.md`](customer/checkout-order.md) | Order - Checkout Order | `POST /api/v1/orders` (Confirmed) |
| [`customer/confirm-order-received.md`](customer/confirm-order-received.md) | Order - Confirm Order Received | `PATCH /api/v1/orders/{id}/received` (Confirmed) |
| [`customer/create-product-review.md`](customer/create-product-review.md) | Review - Create Product Review | `POST /api/v1/products/{productId}/reviews` (Confirmed) |
| [`customer/login.md`](customer/login.md) | Auth - Customer Login | `POST /api/v1/auth/login` (Confirmed) |
| [`customer/logout.md`](customer/logout.md) | Auth - Logout | `POST /api/v1/auth/logout` (Confirmed) |
| [`customer/manage-cart.md`](customer/manage-cart.md) | Cart - Manage Cart | `GET /api/v1/cart; POST/PATCH/DELETE /api/v1/cart/items` (Confirmed) |
| [`customer/pay-for-order-cod.md`](customer/pay-for-order-cod.md) | Payment - Pay for Order (COD) | `No customer COD endpoint is defined; delivery-result trigger is TBD` (TBD) |
| [`customer/pay-for-order-vnpay.md`](customer/pay-for-order-vnpay.md) | Payment - Pay for Order (VNPAY) | `POST /api/v1/payments/vnpay/create`, `GET /api/v1/payments/vnpay/ipn`, `GET /api/v1/payments/vnpay/return` (Confirmed) |
| [`customer/remove-cart-item.md`](customer/remove-cart-item.md) | Cart - Remove Cart Item | `DELETE /api/v1/cart/items/{id}` (Confirmed) |
| [`customer/search-and-filter-products.md`](customer/search-and-filter-products.md) | Catalog - Search and Filter Products | `GET /api/v1/products (query/filter parameters per catalog contract)` (Confirmed) |
| [`customer/update-cart-item.md`](customer/update-cart-item.md) | Cart - Update Cart Item | `PATCH /api/v1/cart/items/{id}` (Confirmed) |
| [`customer/use-ai-product-search.md`](customer/use-ai-product-search.md) | AI - Search Products with AI Assistant | `POST /api/v1/ai/ask` (Planned) |
| [`customer/view-cart.md`](customer/view-cart.md) | Cart - View Cart | `GET /api/v1/cart` (Confirmed) |
| [`customer/view-my-orders.md`](customer/view-my-orders.md) | Order - View My Orders | `GET /api/v1/orders and GET /api/v1/orders/{id}` (Confirmed) |
| [`customer/view-my-shipments.md`](customer/view-my-shipments.md) | Shipment - View My Shipments | `GET /api/v1/orders/{id}` (Confirmed) |
| [`customer/view-product-details.md`](customer/view-product-details.md) | Catalog - View Product Details | `GET /api/v1/products/{id}` (Confirmed) |

### Seller

| Diagram | Use case | API route (status) |
|---|---|---|
| [`seller/cancel-order-item.md`](seller/cancel-order-item.md) | Order - Cancel Order Item | `PATCH /api/v1/seller/order-items/{id}/cancel` (Predicted) |
| [`seller/chat-with-customer.md`](seller/chat-with-customer.md) | Chat - Chat with Customer | `WebSocket /ws/chat (message frames are phased)` (Phased) |
| [`seller/choose-delivery-method.md`](seller/choose-delivery-method.md) | Shipment - Choose Delivery Method | `POST /api/v1/seller/shipments (deliveryMethod is part of the request)` (Predicted) |
| [`seller/confirm-order-item.md`](seller/confirm-order-item.md) | Order - Confirm Order Item | `PATCH /api/v1/seller/order-items/{id}/confirm` (Predicted) |
| [`seller/create-product.md`](seller/create-product.md) | Catalog - Create Product | `POST /api/v1/seller/products` (Confirmed) |
| [`seller/create-shipment.md`](seller/create-shipment.md) | Shipment - Create Shipment | `POST /api/v1/seller/shipments` (Predicted) |
| [`seller/delete-or-unlist-product.md`](seller/delete-or-unlist-product.md) | Catalog - Delete or Unlist Product | `DELETE /api/v1/seller/products/{id} (soft-delete/unlist)` (Confirmed) |
| [`seller/login.md`](seller/login.md) | Auth - Seller Login | `POST /api/v1/auth/login` (Confirmed) |
| [`seller/manage-own-shipments.md`](seller/manage-own-shipments.md) | Shipment - Manage Own Shipments | `GET /api/v1/seller/shipments; PATCH /api/v1/seller/shipments/{id}/status` (Predicted) |
| [`seller/manage-products.md`](seller/manage-products.md) | Catalog - Manage Products | `GET seller products route TBD; POST/PATCH/DELETE /api/v1/seller/products` (Mixed: product reads TBD + confirmed writes) |
| [`seller/process-own-order-items.md`](seller/process-own-order-items.md) | Order - Process Own Order Items | `GET /api/v1/seller/order-items; PATCH confirm/cancel` (Predicted) |
| [`seller/update-product-stock.md`](seller/update-product-stock.md) | Catalog - Update Product Stock | `PATCH /api/v1/seller/products/{id}/stock` (Confirmed) |
| [`seller/update-product.md`](seller/update-product.md) | Catalog - Update Product | `PATCH /api/v1/seller/products/{id}` (Confirmed) |
| [`seller/update-shipment-status.md`](seller/update-shipment-status.md) | Shipment - Update Shipment Status | `PATCH /api/v1/seller/shipments/{id}/status` (Predicted) |
| [`seller/view-own-order-items.md`](seller/view-own-order-items.md) | Order - View Own Order Items | `GET /api/v1/seller/order-items` (Predicted) |
| [`seller/view-own-products.md`](seller/view-own-products.md) | Catalog - View Own Products | `No seller product-list endpoint is defined; route TBD` (TBD) |
| [`seller/view-own-shipments.md`](seller/view-own-shipments.md) | Shipment - View Own Shipments | `GET /api/v1/seller/shipments` (Predicted) |

### Admin

| Diagram | Use case | API route (status) |
|---|---|---|
| [`admin/approve-or-suspend-seller.md`](admin/approve-or-suspend-seller.md) | User - Approve or Suspend Seller | `PATCH /api/v1/admin/sellers/{id}/approve or /suspend` (Confirmed) |
| [`admin/approve-refund.md`](admin/approve-refund.md) | Payment - Approve Refund | `No refund endpoint is defined; route TBD` (TBD) |
| [`admin/configure-platform-delivery.md`](admin/configure-platform-delivery.md) | Shipment - Manage Carriers | `GET`/`POST /api/v1/admin/carriers`, `PATCH /api/v1/admin/carriers/{id}` (Confirmed) |
| [`admin/create-category.md`](admin/create-category.md) | Category - Create Category | `POST /api/v1/admin/categories` (Predicted) |
| [`admin/create-user.md`](admin/create-user.md) | User - Create User | `POST /api/v1/admin/users` (Confirmed) |
| [`admin/create-voucher.md`](admin/create-voucher.md) | Voucher - Create Voucher | `POST /api/v1/admin/vouchers` (Confirmed) |
| [`admin/delete-category.md`](admin/delete-category.md) | Category - Delete Category | `DELETE /api/v1/admin/categories/{id}` (Predicted) |
| [`admin/delete-or-expire-voucher.md`](admin/delete-or-expire-voucher.md) | Voucher - Delete or Expire Voucher | `DELETE /api/v1/admin/vouchers/{code} (expires the voucher)` (Confirmed) |
| [`admin/disable-or-delete-user.md`](admin/disable-or-delete-user.md) | User - Disable or Delete User | `PATCH /api/v1/admin/users/{id} or DELETE /api/v1/admin/users/{id}` (Confirmed) |
| [`admin/login.md`](admin/login.md) | Auth - Admin Login | `POST /api/v1/auth/login` (Confirmed) |
| [`admin/manage-categories.md`](admin/manage-categories.md) | Category - Manage Categories | `GET /api/v1/categories; POST/PATCH/DELETE /api/v1/admin/categories` (Mixed: confirmed public read + predicted admin writes) |
| [`admin/manage-users.md`](admin/manage-users.md) | User - Manage Users | `GET/POST/PATCH/DELETE /api/v1/admin/users` (Confirmed) |
| [`admin/manage-vouchers.md`](admin/manage-vouchers.md) | Voucher - Manage Vouchers | `GET/POST/PATCH/DELETE /api/v1/admin/vouchers` (Confirmed) |
| [`admin/moderate-reviews-and-violations.md`](admin/moderate-reviews-and-violations.md) | Review - Moderate Reviews and Violations | `No admin moderation endpoint exists in the current MVP contract` (Out of current API scope) |
| [`admin/monitor-marketplace-operations.md`](admin/monitor-marketplace-operations.md) | Admin - Monitor Marketplace Operations | `No monitoring endpoint is defined; route TBD` (TBD) |
| [`admin/update-category.md`](admin/update-category.md) | Category - Update Category | `PATCH /api/v1/admin/categories/{id}` (Predicted) |
| [`admin/update-user.md`](admin/update-user.md) | User - Update User | `PATCH /api/v1/admin/users/{id}` (Confirmed) |
| [`admin/update-voucher.md`](admin/update-voucher.md) | Voucher - Update Voucher | `PATCH /api/v1/admin/vouchers/{code}` (Confirmed) |
| [`admin/view-categories.md`](admin/view-categories.md) | Category - View Categories | `GET /api/v1/categories (public route; admin-specific listing is not specified)` (Confirmed public read) |
| [`admin/view-users.md`](admin/view-users.md) | User - View Users | `GET /api/v1/admin/users` (Confirmed) |
| [`admin/view-vouchers.md`](admin/view-vouchers.md) | Voucher - View Vouchers | `GET /api/v1/admin/vouchers` (Confirmed) |

### Integrations

| Diagram | Integration | API route (status) |
|---|---|---|
| [`integrations/vnpay-payment.md`](integrations/vnpay-payment.md) | VNPAY checkout, IPN, Return URL and 24-hour expiry | `POST /api/v1/orders`, `GET /api/v1/payments/vnpay/ipn`, `GET /api/v1/payments/vnpay/return` (Confirmed) |

## Related documents

- [Activity diagrams](../activity-diagram/README.md) — business flow per use case.
- [Use-case specifications](../usecase-specification/) — source of the use cases.
- [Feature API contracts](../features/) — route and status source.
- [System overview](../SYSTEM_OVERVIEW.md) — overall target architecture.
