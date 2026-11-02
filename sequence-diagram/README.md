# Sequence Diagrams

This folder contains one technical Mermaid sequence diagram per use case in
[`usecase-specification/`](../usecase-specification/) (69 use cases), plus one supplemental VNPAY
integration diagram. Unlike the activity diagrams, these diagrams are not only business flows:
they trace the request through the real backend layers.

## Request path

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

> Every diagram was written from the backend code: controller, service and repository names are the real classes and every route is implemented.

## Files

### Guest

| Diagram | Use case | API route (status) |
|---|---|---|
| [`guest/browse-public-catalog.md`](guest/browse-public-catalog.md) | Catalog - Browse Public Catalog | `GET /api/v1/products, GET /api/v1/categories` (Confirmed) |
| [`guest/login.md`](guest/login.md) | Auth - Guest Login | `POST /api/v1/auth/login` (Confirmed) |
| [`guest/register-account.md`](guest/register-account.md) | Auth - Register Account | `POST /api/v1/auth/register` (Confirmed) |
| [`guest/reset-password.md`](guest/reset-password.md) | Auth - Reset Password | `POST /api/v1/auth/password/forgot, POST /api/v1/auth/password/reset` (Confirmed) |
| [`guest/search-and-filter-products.md`](guest/search-and-filter-products.md) | Catalog - Search and Filter Products | `GET /api/v1/products?q=&categoryId=&sellerId=&minPrice=&maxPrice=&minRating=&sort=` (Confirmed) |
| [`guest/use-ai-product-search.md`](guest/use-ai-product-search.md) | AI - Search Products with AI Assistant | `POST /api/v1/ai/ask` (Confirmed) |
| [`guest/view-product-details.md`](guest/view-product-details.md) | Catalog - View Product Details | `GET /api/v1/products/{id}` (Confirmed) |
| [`guest/view-product-reviews.md`](guest/view-product-reviews.md) | Review - View Product Reviews | `GET /api/v1/products/{productId}/reviews` (Confirmed) |
| [`guest/view-seller-shop.md`](guest/view-seller-shop.md) | Catalog - View Seller Shop | `GET /api/v1/sellers/{id}, GET /api/v1/products?sellerId={id}` (Confirmed) |

### Customer

| Diagram | Use case | API route (status) |
|---|---|---|
| [`customer/add-cart-item.md`](customer/add-cart-item.md) | Cart - Add Cart Item | `POST /api/v1/cart/items` (Confirmed) |
| [`customer/apply-voucher.md`](customer/apply-voucher.md) | Voucher - Apply Voucher | `GET /api/v1/vouchers/{code}/validate` (Confirmed) |
| [`customer/browse-catalog.md`](customer/browse-catalog.md) | Catalog - Browse Catalog | `GET /api/v1/products, GET /api/v1/categories` (Confirmed) |
| [`customer/change-password.md`](customer/change-password.md) | Auth - Change Password | `PATCH /api/v1/users/me/password` (Confirmed) |
| [`customer/chat-with-seller.md`](customer/chat-with-seller.md) | Chat - Chat with Seller | `GET/POST /api/v1/conversations, GET/POST /api/v1/conversations/{id}/messages, PATCH /api/v1/conversations/{id}/read, WebSocket /ws/chat` (Confirmed) |
| [`customer/checkout-order.md`](customer/checkout-order.md) | Order - Checkout Order | `POST /api/v1/orders` (Confirmed) |
| [`customer/confirm-order-received.md`](customer/confirm-order-received.md) | Order - Confirm Order Received | `PATCH /api/v1/orders/{id}/received` (Confirmed) |
| [`customer/create-product-review.md`](customer/create-product-review.md) | Review - Create Product Review | `POST /api/v1/products/{productId}/reviews` (Confirmed) |
| [`customer/login.md`](customer/login.md) | Auth - Customer Login | `POST /api/v1/auth/login` (Confirmed) |
| [`customer/logout.md`](customer/logout.md) | Auth - Logout | `POST /api/v1/auth/logout` (Confirmed) |
| [`customer/manage-cart.md`](customer/manage-cart.md) | Cart - Manage Cart | `GET /api/v1/cart, POST/PATCH/DELETE /api/v1/cart/items` (Confirmed) |
| [`customer/manage-profile.md`](customer/manage-profile.md) | User - Manage Profile | `GET /api/v1/users/me, PATCH /api/v1/users/me` (Confirmed) |
| [`customer/pay-for-order-cod.md`](customer/pay-for-order-cod.md) | Payment - Pay for Order (COD) | `POST /api/v1/orders (paymentMethod = COD), PATCH /api/v1/seller/shipments/{id}/status` (Confirmed) |
| [`customer/pay-for-order-vnpay.md`](customer/pay-for-order-vnpay.md) | Payment - Pay for Order (VNPAY) | `POST /api/v1/payments/vnpay/create`, `GET /api/v1/payments/vnpay/ipn`, `GET /api/v1/payments/vnpay/return` (Confirmed) |
| [`customer/remove-cart-item.md`](customer/remove-cart-item.md) | Cart - Remove Cart Item | `DELETE /api/v1/cart/items/{id}` (Confirmed) |
| [`customer/search-and-filter-products.md`](customer/search-and-filter-products.md) | Catalog - Search and Filter Products | `GET /api/v1/products?q=&categoryId=&sellerId=&minPrice=&maxPrice=&minRating=&sort=` (Confirmed) |
| [`customer/update-cart-item.md`](customer/update-cart-item.md) | Cart - Update Cart Item | `PATCH /api/v1/cart/items/{id}` (Confirmed) |
| [`customer/use-ai-product-search.md`](customer/use-ai-product-search.md) | AI - Search Products with AI Assistant | `POST /api/v1/ai/ask` (Confirmed) |
| [`customer/view-ai-chat-history.md`](customer/view-ai-chat-history.md) | AI - View AI Chat History | `GET /api/v1/ai/sessions, GET /api/v1/ai/sessions/{id}/messages` (Confirmed) |
| [`customer/view-cart.md`](customer/view-cart.md) | Cart - View Cart | `GET /api/v1/cart` (Confirmed) |
| [`customer/view-my-orders.md`](customer/view-my-orders.md) | Order - View My Orders | `GET /api/v1/orders, GET /api/v1/orders/{id}` (Confirmed) |
| [`customer/view-my-shipments.md`](customer/view-my-shipments.md) | Shipment - View My Shipments | `GET /api/v1/orders/{id}` (Confirmed) |
| [`customer/view-product-details.md`](customer/view-product-details.md) | Catalog - View Product Details | `GET /api/v1/products/{id}` (Confirmed) |
| [`customer/view-product-reviews.md`](customer/view-product-reviews.md) | Review - View Product Reviews | `GET /api/v1/products/{productId}/reviews` (Confirmed) |
| [`customer/view-seller-shop.md`](customer/view-seller-shop.md) | Catalog - View Seller Shop | `GET /api/v1/sellers/{id}, GET /api/v1/products?sellerId={id}` (Confirmed) |

### Seller

| Diagram | Use case | API route (status) |
|---|---|---|
| [`seller/cancel-order-item.md`](seller/cancel-order-item.md) | Order - Cancel Order Item | `PATCH /api/v1/seller/order-items/{id}/cancel` (Confirmed) |
| [`seller/chat-with-customer.md`](seller/chat-with-customer.md) | Chat - Chat with Customer | `GET/POST /api/v1/conversations, GET/POST /api/v1/conversations/{id}/messages, PATCH /api/v1/conversations/{id}/read, WebSocket /ws/chat` (Confirmed) |
| [`seller/choose-delivery-method.md`](seller/choose-delivery-method.md) | Shipment - Choose Delivery Method | `GET /api/v1/seller/carriers, POST /api/v1/seller/shipments (deliveryMethod, carrierId)` (Confirmed) |
| [`seller/confirm-order-item.md`](seller/confirm-order-item.md) | Order - Confirm Order Item | `PATCH /api/v1/seller/order-items/{id}/confirm` (Confirmed) |
| [`seller/create-product.md`](seller/create-product.md) | Catalog - Create Product | `POST /api/v1/seller/products` (Confirmed) |
| [`seller/create-shipment.md`](seller/create-shipment.md) | Shipment - Create Shipment | `POST /api/v1/seller/shipments` (Confirmed) |
| [`seller/login.md`](seller/login.md) | Auth - Seller Login | `POST /api/v1/auth/login` (Confirmed) |
| [`seller/manage-own-shipments.md`](seller/manage-own-shipments.md) | Shipment - Manage Own Shipments | `POST/GET /api/v1/seller/shipments, PATCH /api/v1/seller/shipments/{id}/status` (Confirmed) |
| [`seller/manage-products.md`](seller/manage-products.md) | Catalog - Manage Products | `GET/POST/PATCH/DELETE /api/v1/seller/products` (Confirmed) |
| [`seller/process-own-order-items.md`](seller/process-own-order-items.md) | Order - Process Own Order Items | `GET /api/v1/seller/order-items, PATCH /api/v1/seller/order-items/{id}/confirm, PATCH /api/v1/seller/order-items/{id}/cancel` (Confirmed) |
| [`seller/unlist-product.md`](seller/unlist-product.md) | Catalog - Unlist Product | `DELETE /api/v1/seller/products/{id}` (Confirmed) |
| [`seller/update-product.md`](seller/update-product.md) | Catalog - Update Product | `PATCH /api/v1/seller/products/{id}` (Confirmed) |
| [`seller/update-product-stock.md`](seller/update-product-stock.md) | Catalog - Update Product Stock | `PATCH /api/v1/seller/products/{id}/stock` (Confirmed) |
| [`seller/update-shipment-status.md`](seller/update-shipment-status.md) | Shipment - Update Shipment Status | `PATCH /api/v1/seller/shipments/{id}/status` (Confirmed) |
| [`seller/view-own-order-items.md`](seller/view-own-order-items.md) | Order - View Own Order Items | `GET /api/v1/seller/order-items?status=&orderId=` (Confirmed) |
| [`seller/view-own-products.md`](seller/view-own-products.md) | Catalog - View Own Products | `GET /api/v1/seller/products, GET /api/v1/seller/products/{id}` (Confirmed) |
| [`seller/view-own-shipments.md`](seller/view-own-shipments.md) | Shipment - View Own Shipments | `GET /api/v1/seller/shipments?status=&orderId=` (Confirmed) |

### Admin

| Diagram | Use case | API route (status) |
|---|---|---|
| [`admin/approve-or-suspend-seller.md`](admin/approve-or-suspend-seller.md) | Admin - Approve or Suspend Seller | `PATCH /api/v1/admin/sellers/{id}/approve, PATCH /api/v1/admin/sellers/{id}/suspend` (Confirmed) |
| [`admin/create-category.md`](admin/create-category.md) | Category - Create Category | `POST /api/v1/admin/categories` (Confirmed) |
| [`admin/create-user.md`](admin/create-user.md) | Admin - Create User | `POST /api/v1/admin/users` (Confirmed) |
| [`admin/create-voucher.md`](admin/create-voucher.md) | Voucher - Create Voucher | `POST /api/v1/admin/vouchers` (Confirmed) |
| [`admin/delete-category.md`](admin/delete-category.md) | Category - Delete Category | `DELETE /api/v1/admin/categories/{id}` (Confirmed) |
| [`admin/disable-user.md`](admin/disable-user.md) | Admin - Disable User | `DELETE /api/v1/admin/users/{id}` (Confirmed) |
| [`admin/expire-voucher.md`](admin/expire-voucher.md) | Voucher - Expire Voucher | `DELETE /api/v1/admin/vouchers/{code}` (Confirmed) |
| [`admin/login.md`](admin/login.md) | Auth - Admin Login | `POST /api/v1/auth/login` (Confirmed) |
| [`admin/manage-carriers.md`](admin/manage-carriers.md) | Shipment - Manage Carriers | `GET`/`POST /api/v1/admin/carriers`, `PATCH /api/v1/admin/carriers/{id}` (Confirmed) |
| [`admin/manage-categories.md`](admin/manage-categories.md) | Category - Manage Categories | `GET /api/v1/categories, POST/PATCH/DELETE /api/v1/admin/categories` (Confirmed) |
| [`admin/manage-users.md`](admin/manage-users.md) | Admin - Manage Users | `GET/POST/PATCH/DELETE /api/v1/admin/users, PATCH /api/v1/admin/sellers/{id}/approve or /suspend` (Confirmed) |
| [`admin/manage-vouchers.md`](admin/manage-vouchers.md) | Voucher - Manage Vouchers | `GET/POST/PATCH/DELETE /api/v1/admin/vouchers` (Confirmed) |
| [`admin/update-category.md`](admin/update-category.md) | Category - Update Category | `PATCH /api/v1/admin/categories/{id}` (Confirmed) |
| [`admin/update-user.md`](admin/update-user.md) | Admin - Update User | `PATCH /api/v1/admin/users/{id}` (Confirmed) |
| [`admin/update-voucher.md`](admin/update-voucher.md) | Voucher - Update Voucher | `PATCH /api/v1/admin/vouchers/{code}` (Confirmed) |
| [`admin/view-categories.md`](admin/view-categories.md) | Category - View Categories | `GET /api/v1/categories` (Confirmed) |
| [`admin/view-users.md`](admin/view-users.md) | Admin - View Users | `GET /api/v1/admin/users?role=&search=, GET /api/v1/admin/users/{id}` (Confirmed) |
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
