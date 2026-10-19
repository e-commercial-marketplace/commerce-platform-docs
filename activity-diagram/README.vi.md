# Sơ đồ hoạt động (Activity Diagram)

Thư mục này chứa mỗi use case trong [`usecase-specification/`](../usecase-specification/) một sơ đồ
hoạt động PlantUML. Mỗi sơ đồ là góc nhìn luồng xử lý của spec tương ứng: luồng chính, luồng thay
thế, luồng ngoại lệ và tiền/hậu điều kiện, được vẽ bằng theme PlantUML của Visual Paradigm với hai
swimlane (tác nhân chính và hệ thống).

Các sơ đồ là góc nhìn tài liệu. Chúng không thay thế use-case spec, API contract hay mã nguồn
triển khai.

## Tệp

### Guest

| Tệp | Use case | Spec |
|---|---|---|
| [`guest/register-customer-account.puml`](guest/register-customer-account.puml) | Đăng ký tài khoản | [`register-customer-account.vi.md`](../usecase-specification/guest/register-customer-account.vi.md) |
| [`guest/login.puml`](guest/login.puml) | Đăng nhập (Guest) | [`login.vi.md`](../usecase-specification/guest/login.vi.md) |
| [`guest/browse-public-catalog.puml`](guest/browse-public-catalog.puml) | Duyệt catalog công khai | [`browse-public-catalog.vi.md`](../usecase-specification/guest/browse-public-catalog.vi.md) |
| [`guest/search-and-filter-products.puml`](guest/search-and-filter-products.puml) | Tìm kiếm và lọc sản phẩm | [`search-and-filter-products.vi.md`](../usecase-specification/guest/search-and-filter-products.vi.md) |
| [`guest/view-product-details.puml`](guest/view-product-details.puml) | Xem chi tiết sản phẩm | [`view-product-details.vi.md`](../usecase-specification/guest/view-product-details.vi.md) |
| [`guest/use-ai-product-search.puml`](guest/use-ai-product-search.puml) | Tìm kiếm sản phẩm bằng trợ lý AI | [`use-ai-product-search.vi.md`](../usecase-specification/guest/use-ai-product-search.vi.md) |

### Customer

| Tệp | Use case | Spec |
|---|---|---|
| [`customer/login.puml`](customer/login.puml) | Đăng nhập (Customer) | [`login.vi.md`](../usecase-specification/customer/login.vi.md) |
| [`customer/browse-catalog.puml`](customer/browse-catalog.puml) | Duyệt catalog | [`browse-catalog.vi.md`](../usecase-specification/customer/browse-catalog.vi.md) |
| [`customer/search-and-filter-products.puml`](customer/search-and-filter-products.puml) | Tìm kiếm và lọc sản phẩm | [`search-and-filter-products.vi.md`](../usecase-specification/customer/search-and-filter-products.vi.md) |
| [`customer/view-product-details.puml`](customer/view-product-details.puml) | Xem chi tiết sản phẩm | [`view-product-details.vi.md`](../usecase-specification/customer/view-product-details.vi.md) |
| [`customer/use-ai-product-search.puml`](customer/use-ai-product-search.puml) | Tìm kiếm sản phẩm bằng trợ lý AI | [`use-ai-product-search.vi.md`](../usecase-specification/customer/use-ai-product-search.vi.md) |
| [`customer/view-cart.puml`](customer/view-cart.puml) | Xem giỏ hàng | [`view-cart.vi.md`](../usecase-specification/customer/view-cart.vi.md) |
| [`customer/add-cart-item.puml`](customer/add-cart-item.puml) | Thêm sản phẩm vào giỏ hàng | [`add-cart-item.vi.md`](../usecase-specification/customer/add-cart-item.vi.md) |
| [`customer/update-cart-item.puml`](customer/update-cart-item.puml) | Cập nhật sản phẩm trong giỏ hàng | [`update-cart-item.vi.md`](../usecase-specification/customer/update-cart-item.vi.md) |
| [`customer/remove-cart-item.puml`](customer/remove-cart-item.puml) | Xóa sản phẩm khỏi giỏ hàng | [`remove-cart-item.vi.md`](../usecase-specification/customer/remove-cart-item.vi.md) |
| [`customer/manage-cart.puml`](customer/manage-cart.puml) | Quản lý giỏ hàng | [`manage-cart.vi.md`](../usecase-specification/customer/manage-cart.vi.md) |
| [`customer/checkout-order.puml`](customer/checkout-order.puml) | Đặt hàng | [`checkout-order.vi.md`](../usecase-specification/customer/checkout-order.vi.md) |
| [`customer/pay-for-order-cod.puml`](customer/pay-for-order-cod.puml) | Thanh toán khi nhận hàng (COD) | [`pay-for-order-cod.vi.md`](../usecase-specification/customer/pay-for-order-cod.vi.md) |
| [`customer/view-my-orders.puml`](customer/view-my-orders.puml) | Xem đơn hàng của tôi | [`view-my-orders.vi.md`](../usecase-specification/customer/view-my-orders.vi.md) |
| [`customer/view-my-shipments.puml`](customer/view-my-shipments.puml) | Xem các đợt giao hàng của tôi | [`view-my-shipments.vi.md`](../usecase-specification/customer/view-my-shipments.vi.md) |
| [`customer/create-product-review.puml`](customer/create-product-review.puml) | Tạo đánh giá sản phẩm | [`create-product-review.vi.md`](../usecase-specification/customer/create-product-review.vi.md) |
| [`customer/chat-with-seller.puml`](customer/chat-with-seller.puml) | Trò chuyện với người bán | [`chat-with-seller.vi.md`](../usecase-specification/customer/chat-with-seller.vi.md) |

### Seller

| Tệp | Use case | Spec |
|---|---|---|
| [`seller/login.puml`](seller/login.puml) | Đăng nhập (Seller) | [`login.vi.md`](../usecase-specification/seller/login.vi.md) |
| [`seller/manage-products.puml`](seller/manage-products.puml) | Quản lý sản phẩm | [`manage-products.vi.md`](../usecase-specification/seller/manage-products.vi.md) |
| [`seller/create-product.puml`](seller/create-product.puml) | Tạo sản phẩm | [`create-product.vi.md`](../usecase-specification/seller/create-product.vi.md) |
| [`seller/update-product.puml`](seller/update-product.puml) | Cập nhật sản phẩm | [`update-product.vi.md`](../usecase-specification/seller/update-product.vi.md) |
| [`seller/update-product-stock.puml`](seller/update-product-stock.puml) | Cập nhật tồn kho sản phẩm | [`update-product-stock.vi.md`](../usecase-specification/seller/update-product-stock.vi.md) |
| [`seller/delete-or-unlist-product.puml`](seller/delete-or-unlist-product.puml) | Xóa hoặc ẩn sản phẩm | [`delete-or-unlist-product.vi.md`](../usecase-specification/seller/delete-or-unlist-product.vi.md) |
| [`seller/view-own-products.puml`](seller/view-own-products.puml) | Xem sản phẩm của tôi | [`view-own-products.vi.md`](../usecase-specification/seller/view-own-products.vi.md) |
| [`seller/view-own-order-items.puml`](seller/view-own-order-items.puml) | Xem mặt hàng trong đơn hàng của tôi | [`view-own-order-items.vi.md`](../usecase-specification/seller/view-own-order-items.vi.md) |
| [`seller/confirm-order-item.puml`](seller/confirm-order-item.puml) | Xác nhận mặt hàng trong đơn hàng | [`confirm-order-item.vi.md`](../usecase-specification/seller/confirm-order-item.vi.md) |
| [`seller/cancel-order-item.puml`](seller/cancel-order-item.puml) | Hủy mặt hàng trong đơn hàng | [`cancel-order-item.vi.md`](../usecase-specification/seller/cancel-order-item.vi.md) |
| [`seller/process-own-order-items.puml`](seller/process-own-order-items.puml) | Xử lý mặt hàng trong đơn hàng của tôi | [`process-own-order-items.vi.md`](../usecase-specification/seller/process-own-order-items.vi.md) |
| [`seller/choose-delivery-method.puml`](seller/choose-delivery-method.puml) | Chọn phương thức giao hàng | [`choose-delivery-method.vi.md`](../usecase-specification/seller/choose-delivery-method.vi.md) |
| [`seller/create-shipment.puml`](seller/create-shipment.puml) | Tạo đợt giao hàng | [`create-shipment.vi.md`](../usecase-specification/seller/create-shipment.vi.md) |
| [`seller/update-shipment-status.puml`](seller/update-shipment-status.puml) | Cập nhật trạng thái giao hàng | [`update-shipment-status.vi.md`](../usecase-specification/seller/update-shipment-status.vi.md) |
| [`seller/view-own-shipments.puml`](seller/view-own-shipments.puml) | Xem các đợt giao hàng của tôi | [`view-own-shipments.vi.md`](../usecase-specification/seller/view-own-shipments.vi.md) |
| [`seller/manage-own-shipments.puml`](seller/manage-own-shipments.puml) | Quản lý các đợt giao hàng của tôi | [`manage-own-shipments.vi.md`](../usecase-specification/seller/manage-own-shipments.vi.md) |
| [`seller/chat-with-customer.puml`](seller/chat-with-customer.puml) | Trò chuyện với khách hàng | [`chat-with-customer.vi.md`](../usecase-specification/seller/chat-with-customer.vi.md) |

### Admin

| Tệp | Use case | Spec |
|---|---|---|
| [`admin/login.puml`](admin/login.puml) | Đăng nhập (Admin) | [`login.vi.md`](../usecase-specification/admin/login.vi.md) |
| [`admin/manage-users.puml`](admin/manage-users.puml) | Quản lý người dùng | [`manage-users.vi.md`](../usecase-specification/admin/manage-users.vi.md) |
| [`admin/create-user.puml`](admin/create-user.puml) | Tạo người dùng | [`create-user.vi.md`](../usecase-specification/admin/create-user.vi.md) |
| [`admin/update-user.puml`](admin/update-user.puml) | Cập nhật thông tin người dùng | [`update-user.vi.md`](../usecase-specification/admin/update-user.vi.md) |
| [`admin/disable-or-delete-user.puml`](admin/disable-or-delete-user.puml) | Vô hiệu hóa hoặc xóa người dùng | [`disable-or-delete-user.vi.md`](../usecase-specification/admin/disable-or-delete-user.vi.md) |
| [`admin/view-users.puml`](admin/view-users.puml) | Xem người dùng | [`view-users.vi.md`](../usecase-specification/admin/view-users.vi.md) |
| [`admin/approve-or-suspend-seller.puml`](admin/approve-or-suspend-seller.puml) | Duyệt hoặc tạm ngưng người bán | [`approve-or-suspend-seller.vi.md`](../usecase-specification/admin/approve-or-suspend-seller.vi.md) |
| [`admin/manage-categories.puml`](admin/manage-categories.puml) | Quản lý danh mục | [`manage-categories.vi.md`](../usecase-specification/admin/manage-categories.vi.md) |
| [`admin/create-category.puml`](admin/create-category.puml) | Tạo danh mục | [`create-category.vi.md`](../usecase-specification/admin/create-category.vi.md) |
| [`admin/update-category.puml`](admin/update-category.puml) | Cập nhật danh mục | [`update-category.vi.md`](../usecase-specification/admin/update-category.vi.md) |
| [`admin/delete-category.puml`](admin/delete-category.puml) | Xóa danh mục | [`delete-category.vi.md`](../usecase-specification/admin/delete-category.vi.md) |
| [`admin/view-categories.puml`](admin/view-categories.puml) | Xem danh mục | [`view-categories.vi.md`](../usecase-specification/admin/view-categories.vi.md) |
| [`admin/manage-vouchers.puml`](admin/manage-vouchers.puml) | Quản lý mã giảm giá | [`manage-vouchers.vi.md`](../usecase-specification/admin/manage-vouchers.vi.md) |
| [`admin/create-voucher.puml`](admin/create-voucher.puml) | Tạo mã giảm giá | [`create-voucher.vi.md`](../usecase-specification/admin/create-voucher.vi.md) |
| [`admin/update-voucher.puml`](admin/update-voucher.puml) | Cập nhật mã giảm giá | [`update-voucher.vi.md`](../usecase-specification/admin/update-voucher.vi.md) |
| [`admin/delete-or-expire-voucher.puml`](admin/delete-or-expire-voucher.puml) | Xóa hoặc kết thúc mã giảm giá | [`delete-or-expire-voucher.vi.md`](../usecase-specification/admin/delete-or-expire-voucher.vi.md) |
| [`admin/view-vouchers.puml`](admin/view-vouchers.puml) | Xem mã giảm giá | [`view-vouchers.vi.md`](../usecase-specification/admin/view-vouchers.vi.md) |
| [`admin/moderate-reviews-and-violations.puml`](admin/moderate-reviews-and-violations.puml) | Kiểm duyệt đánh giá và vi phạm | [`moderate-reviews-and-violations.vi.md`](../usecase-specification/admin/moderate-reviews-and-violations.vi.md) |
| [`admin/approve-refund.puml`](admin/approve-refund.puml) | Duyệt hoàn tiền | [`approve-refund.vi.md`](../usecase-specification/admin/approve-refund.vi.md) |
| [`admin/configure-platform-delivery.puml`](admin/configure-platform-delivery.puml) | Cấu hình giao hàng nền tảng | [`configure-platform-delivery.vi.md`](../usecase-specification/admin/configure-platform-delivery.vi.md) |
| [`admin/monitor-marketplace-operations.puml`](admin/monitor-marketplace-operations.puml) | Theo dõi hoạt động marketplace | [`monitor-marketplace-operations.vi.md`](../usecase-specification/admin/monitor-marketplace-operations.vi.md) |

## Cách render

- PlantUML CLI: `java -jar plantuml.jar -tpng activity-diagram/guest/login.puml` (dùng `-tsvg` để
  xuất SVG).
- VS Code: cài extension PlantUML rồi mở file `.puml` (`Alt+D` để xem trước).
- Mọi file đều include theme
  `https://static.visual-paradigm.com/web/resources/plantuml-stdlib/themes/vp.puml`, nên lần render
  đầu cần internet; sau đó PlantUML cache lại theme.
- Chỉ commit file nguồn `.puml`; PNG/SVG sinh ra chỉ là artifact tạm.

## Quy ước

- Mọi file mở đầu bằng include theme Visual Paradigm và cùng một khối `skinparam`: activity màu
  xanh (`#7DD3FC`, viền `#2563EB`), node start đen, node end trắng, mũi tên và viền swimlane 1.5px
  `#222222`.
- Title theo định dạng `<area> - <use case>`, với `<area>` thuộc: Auth, Catalog, Cart, Order,
  Payment, Shipment, Voucher, Review, Chat, AI, User, Category, Admin.
- Swimlane: tác nhân chính của spec (`Guest`, `Customer`, `Seller` hoặc `Admin`) và `System`.
  Không spec nào có tác nhân phụ. Bốn use case `login` nằm ở lane `Guest` vì tác nhân chính của
  chúng là guest đang chọn tài khoản để đăng nhập.
- Tiền điều kiện và hậu điều kiện là note gắn vào node start (PlantUML bỏ qua note "trôi" đặt trước
  `start`, nên note được đặt ngay sau `start`).
- Các bước luồng chính thành các activity theo đúng thứ tự bảng spec; mỗi validation hoặc business
  rule thành một diamond quyết định với nhánh Yes/No.
- Luồng thay thế có thể phục hồi (spec ghi tác nhân sửa dữ liệu và quay lại bước N) dùng vòng lặp
  `repeat`/`repeat while` bao quanh lần xử lý để không bỏ qua lần thử đầu tiên. Chèn ngắt dòng trong
  nhãn retry (ví dụ `Submit\nagain?`) để đường nối loop đi giữa hai dòng chữ, không gạch ngang nhãn.
  Lỗi kết thúc ("kết thúc lần xử lý này") và các lần từ chối kết thúc bằng `stop`.
- Nhãn quyết định retry cần ngắn gọn; actor đã rõ qua swimlane, câu hỏi dài sẽ bị chật trong diamond.
- Luồng ngoại lệ là nhánh `stop` kết thúc; sơ đồ không tự thêm cách phục hồi mà spec không mô tả.
- Chữ trong sơ đồ là tiếng Anh; spec giữ tiếng Việt. Khi spec đổi, cập nhật sơ đồ tương ứng.

Tài liệu liên quan: [`usecase-specification/`](../usecase-specification/) (spec nguồn),
[`usecase-diagram/`](../usecase-diagram/README.vi.md) (sơ đồ use-case theo tác nhân), và ranh giới
MVP trong `PRD.vi.md` cùng SRS.
