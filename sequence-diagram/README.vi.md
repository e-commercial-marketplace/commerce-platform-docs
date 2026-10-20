# Sơ đồ tuần tự

Thư mục này chứa mỗi use case trong [`usecase-specification/`](../usecase-specification/) một
sequence diagram kỹ thuật bằng Mermaid (60 use case), kèm một sơ đồ bổ sung cho tích hợp VNPAY.
Khác với activity diagram, các sơ đồ này không chỉ mô tả luồng nghiệp vụ mà thể hiện request đi
qua các tầng backend thật.

## Luồng request theo kiến trúc mục tiêu

```text
Client -> Spring Security -> FeatureController -> FeatureService -> FeatureRepository -> PostgreSQL
```

- `FeatureService` là tầng service của feature: nhận call từ controller, xử lý business rule và
  đọc/ghi qua repository của feature. Mỗi sơ đồ chỉ vẽ một participant `Service`; phần tách
  interface/implementation trong Spring là chi tiết nội bộ, không vẽ thành hop riêng.
- Gọi giữa các feature dùng service của feature sở hữu (ví dụ order service gọi `CartService`);
  không truy cập trực tiếp repository hay entity của feature khác.
- External provider (`VNPAY`, LLM provider, email) chỉ xuất hiện trong sơ đồ mà contract hoặc spec
  yêu cầu; secret của provider luôn nằm ở backend.
- `GlobalExceptionHandler` là luồng lỗi dùng chung; trả `ApiResponse<T>` lỗi theo contract và không
  lộ chi tiết nội bộ.

> **Thiết kế mục tiêu:** backend scaffold hiện chưa có các tầng controller/service/repository.
> Mỗi sơ đồ ghi rõ trạng thái endpoint (Confirmed / Predicted / TBD / Phased) cạnh route.
> Khi chưa có contract, sơ đồ dừng lại thay vì tự đặt route hoặc thêm external call.

## Danh sách file

### Khách chưa đăng nhập (Guest)

| Diagram | Use case | API route (status) |
|---|---|---|
| [`guest/browse-public-catalog.md`](guest/browse-public-catalog.md) | Catalog - Browse Public Catalog | `GET /api/v1/products and GET /api/v1/categories` (Confirmed) |
| [`guest/login.md`](guest/login.md) | Auth - Guest Login | `POST /api/v1/auth/login` (Confirmed) |
| [`guest/register-customer-account.md`](guest/register-customer-account.md) | Auth - Register Customer Account | `POST /api/v1/auth/register` (Confirmed) |
| [`guest/search-and-filter-products.md`](guest/search-and-filter-products.md) | Catalog - Search and Filter Products | `GET /api/v1/products (query/filter parameters per catalog contract)` (Confirmed) |
| [`guest/use-ai-product-search.md`](guest/use-ai-product-search.md) | AI - Search Products with AI Assistant | `POST /api/v1/ai/ask` (Planned) |
| [`guest/view-product-details.md`](guest/view-product-details.md) | Catalog - View Product Details | `GET /api/v1/products/{id}` (Confirmed) |

### Khách hàng (Customer)

| Diagram | Use case | API route (status) |
|---|---|---|
| [`customer/add-cart-item.md`](customer/add-cart-item.md) | Cart - Add Cart Item | `POST /api/v1/cart/items` (Confirmed) |
| [`customer/browse-catalog.md`](customer/browse-catalog.md) | Catalog - Browse Catalog | `GET /api/v1/products and GET /api/v1/categories` (Confirmed) |
| [`customer/chat-with-seller.md`](customer/chat-with-seller.md) | Chat - Chat with Seller | `WebSocket /ws/chat (message frames are phased)` (Phased) |
| [`customer/checkout-order.md`](customer/checkout-order.md) | Order - Checkout Order | `POST /api/v1/orders` (Confirmed) |
| [`customer/create-product-review.md`](customer/create-product-review.md) | Review - Create Product Review | `POST /api/v1/products/{productId}/reviews` (Confirmed) |
| [`customer/login.md`](customer/login.md) | Auth - Customer Login | `POST /api/v1/auth/login` (Confirmed) |
| [`customer/manage-cart.md`](customer/manage-cart.md) | Cart - Manage Cart | `GET /api/v1/cart; POST/PATCH/DELETE /api/v1/cart/items` (Confirmed) |
| [`customer/pay-for-order-cod.md`](customer/pay-for-order-cod.md) | Payment - Pay for Order (COD) | `No customer COD endpoint is defined; delivery-result trigger is TBD` (TBD) |
| [`customer/remove-cart-item.md`](customer/remove-cart-item.md) | Cart - Remove Cart Item | `DELETE /api/v1/cart/items/{id}` (Confirmed) |
| [`customer/search-and-filter-products.md`](customer/search-and-filter-products.md) | Catalog - Search and Filter Products | `GET /api/v1/products (query/filter parameters per catalog contract)` (Confirmed) |
| [`customer/update-cart-item.md`](customer/update-cart-item.md) | Cart - Update Cart Item | `PATCH /api/v1/cart/items/{id}` (Confirmed) |
| [`customer/use-ai-product-search.md`](customer/use-ai-product-search.md) | AI - Search Products with AI Assistant | `POST /api/v1/ai/ask` (Planned) |
| [`customer/view-cart.md`](customer/view-cart.md) | Cart - View Cart | `GET /api/v1/cart` (Confirmed) |
| [`customer/view-my-orders.md`](customer/view-my-orders.md) | Order - View My Orders | `GET /api/v1/orders and GET /api/v1/orders/{id}` (Confirmed) |
| [`customer/view-my-shipments.md`](customer/view-my-shipments.md) | Shipment - View My Shipments | `GET /api/v1/orders/{orderId}/shipments` (Predicted) |
| [`customer/view-product-details.md`](customer/view-product-details.md) | Catalog - View Product Details | `GET /api/v1/products/{id}` (Confirmed) |

### Người bán (Seller)

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

### Quản trị viên (Admin)

| Diagram | Use case | API route (status) |
|---|---|---|
| [`admin/approve-or-suspend-seller.md`](admin/approve-or-suspend-seller.md) | User - Approve or Suspend Seller | `PATCH /api/v1/admin/sellers/{id}/approve or /suspend` (Confirmed) |
| [`admin/approve-refund.md`](admin/approve-refund.md) | Payment - Approve Refund | `No refund endpoint is defined; route TBD` (TBD) |
| [`admin/configure-platform-delivery.md`](admin/configure-platform-delivery.md) | Shipment - Configure Platform Delivery | `No endpoint is defined; route TBD` (TBD) |
| [`admin/create-category.md`](admin/create-category.md) | Category - Create Category | `POST /api/v1/admin/categories` (Predicted) |
| [`admin/create-user.md`](admin/create-user.md) | User - Create User | `POST /api/v1/admin/users` (Confirmed) |
| [`admin/create-voucher.md`](admin/create-voucher.md) | Voucher - Create Voucher | `POST /api/v1/admin/vouchers` (Predicted) |
| [`admin/delete-category.md`](admin/delete-category.md) | Category - Delete Category | `DELETE /api/v1/admin/categories/{id}` (Predicted) |
| [`admin/delete-or-expire-voucher.md`](admin/delete-or-expire-voucher.md) | Voucher - Delete or Expire Voucher | `DELETE /api/v1/admin/vouchers/{id} (expires the voucher)` (Predicted) |
| [`admin/disable-or-delete-user.md`](admin/disable-or-delete-user.md) | User - Disable or Delete User | `PATCH /api/v1/admin/users/{id} or DELETE /api/v1/admin/users/{id}` (Confirmed) |
| [`admin/login.md`](admin/login.md) | Auth - Admin Login | `POST /api/v1/auth/login` (Confirmed) |
| [`admin/manage-categories.md`](admin/manage-categories.md) | Category - Manage Categories | `GET /api/v1/categories; POST/PATCH/DELETE /api/v1/admin/categories` (Mixed: confirmed public read + predicted admin writes) |
| [`admin/manage-users.md`](admin/manage-users.md) | User - Manage Users | `GET/POST/PATCH/DELETE /api/v1/admin/users` (Confirmed) |
| [`admin/manage-vouchers.md`](admin/manage-vouchers.md) | Voucher - Manage Vouchers | `GET/POST/PATCH/DELETE /api/v1/admin/vouchers` (Predicted) |
| [`admin/moderate-reviews-and-violations.md`](admin/moderate-reviews-and-violations.md) | Review - Moderate Reviews and Violations | `No admin moderation endpoint exists in the current MVP contract` (Out of current API scope) |
| [`admin/monitor-marketplace-operations.md`](admin/monitor-marketplace-operations.md) | Admin - Monitor Marketplace Operations | `No monitoring endpoint is defined; route TBD` (TBD) |
| [`admin/update-category.md`](admin/update-category.md) | Category - Update Category | `PATCH /api/v1/admin/categories/{id}` (Predicted) |
| [`admin/update-user.md`](admin/update-user.md) | User - Update User | `PATCH /api/v1/admin/users/{id}` (Confirmed) |
| [`admin/update-voucher.md`](admin/update-voucher.md) | Voucher - Update Voucher | `PATCH /api/v1/admin/vouchers/{id}` (Predicted) |
| [`admin/view-categories.md`](admin/view-categories.md) | Category - View Categories | `GET /api/v1/categories (public route; admin-specific listing is not specified)` (Confirmed public read) |
| [`admin/view-users.md`](admin/view-users.md) | User - View Users | `GET /api/v1/admin/users` (Confirmed) |
| [`admin/view-vouchers.md`](admin/view-vouchers.md) | Voucher - View Vouchers | `GET /api/v1/admin/vouchers` (Predicted) |

### Tích hợp

| Diagram | Tích hợp | API route (status) |
|---|---|---|
| [`integrations/vnpay-payment.md`](integrations/vnpay-payment.md) | VNPAY create + callback | `POST /api/v1/payments/vnpay/create`, `POST /api/v1/payments/vnpay/callback` (Predicted) |

## Tài liệu liên quan

- [Activity diagram](../activity-diagram/README.vi.md) — luồng nghiệp vụ từng use case.
- [Đặc tả use case](../usecase-specification/) — nguồn của các use case.
- [Feature API contract](../features/) — nguồn route và status.
- [Tổng quan hệ thống](../SYSTEM_OVERVIEW.vi.md) — kiến trúc mục tiêu tổng thể.
