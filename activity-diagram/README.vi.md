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
| [`guest/browse-public-catalog.puml`](guest/browse-public-catalog.puml) | Duyệt catalog | [`browse-public-catalog.vi.md`](../usecase-specification/guest/browse-public-catalog.vi.md) |
| [`guest/login.puml`](guest/login.puml) | Đăng nhập | [`login.vi.md`](../usecase-specification/guest/login.vi.md) |
| [`guest/register-account.puml`](guest/register-account.puml) | Đăng ký tài khoản | [`register-account.vi.md`](../usecase-specification/guest/register-account.vi.md) |
| [`guest/reset-password.puml`](guest/reset-password.puml) | Đặt lại mật khẩu | [`reset-password.vi.md`](../usecase-specification/guest/reset-password.vi.md) |
| [`guest/search-and-filter-products.puml`](guest/search-and-filter-products.puml) | Tìm kiếm và lọc sản phẩm | [`search-and-filter-products.vi.md`](../usecase-specification/guest/search-and-filter-products.vi.md) |
| [`guest/use-ai-product-search.puml`](guest/use-ai-product-search.puml) | Tìm sản phẩm bằng trợ lý AI | [`use-ai-product-search.vi.md`](../usecase-specification/guest/use-ai-product-search.vi.md) |
| [`guest/view-product-details.puml`](guest/view-product-details.puml) | Xem chi tiết sản phẩm | [`view-product-details.vi.md`](../usecase-specification/guest/view-product-details.vi.md) |
| [`guest/view-product-reviews.puml`](guest/view-product-reviews.puml) | Xem đánh giá sản phẩm | [`view-product-reviews.vi.md`](../usecase-specification/guest/view-product-reviews.vi.md) |
| [`guest/view-seller-shop.puml`](guest/view-seller-shop.puml) | Xem trang shop | [`view-seller-shop.vi.md`](../usecase-specification/guest/view-seller-shop.vi.md) |

### Customer

| Tệp | Use case | Spec |
|---|---|---|
| [`customer/add-cart-item.puml`](customer/add-cart-item.puml) | Thêm sản phẩm vào giỏ | [`add-cart-item.vi.md`](../usecase-specification/customer/add-cart-item.vi.md) |
| [`customer/apply-voucher.puml`](customer/apply-voucher.puml) | Áp dụng voucher | [`apply-voucher.vi.md`](../usecase-specification/customer/apply-voucher.vi.md) |
| [`customer/browse-catalog.puml`](customer/browse-catalog.puml) | Duyệt catalog | [`browse-catalog.vi.md`](../usecase-specification/customer/browse-catalog.vi.md) |
| [`customer/change-password.puml`](customer/change-password.puml) | Đổi mật khẩu | [`change-password.vi.md`](../usecase-specification/customer/change-password.vi.md) |
| [`customer/chat-with-seller.puml`](customer/chat-with-seller.puml) | Trò chuyện với Seller | [`chat-with-seller.vi.md`](../usecase-specification/customer/chat-with-seller.vi.md) |
| [`customer/checkout-order.puml`](customer/checkout-order.puml) | Đặt hàng | [`checkout-order.vi.md`](../usecase-specification/customer/checkout-order.vi.md) |
| [`customer/confirm-order-received.puml`](customer/confirm-order-received.puml) | Xác nhận đã nhận hàng | [`confirm-order-received.vi.md`](../usecase-specification/customer/confirm-order-received.vi.md) |
| [`customer/create-product-review.puml`](customer/create-product-review.puml) | Đánh giá sản phẩm | [`create-product-review.vi.md`](../usecase-specification/customer/create-product-review.vi.md) |
| [`customer/login.puml`](customer/login.puml) | Đăng nhập | [`login.vi.md`](../usecase-specification/customer/login.vi.md) |
| [`customer/logout.puml`](customer/logout.puml) | Đăng xuất | [`logout.vi.md`](../usecase-specification/customer/logout.vi.md) |
| [`customer/manage-cart.puml`](customer/manage-cart.puml) | Quản lý giỏ hàng | [`manage-cart.vi.md`](../usecase-specification/customer/manage-cart.vi.md) |
| [`customer/manage-profile.puml`](customer/manage-profile.puml) | Quản lý hồ sơ | [`manage-profile.vi.md`](../usecase-specification/customer/manage-profile.vi.md) |
| [`customer/pay-for-order-cod.puml`](customer/pay-for-order-cod.puml) | Thanh toán khi nhận hàng (COD) | [`pay-for-order-cod.vi.md`](../usecase-specification/customer/pay-for-order-cod.vi.md) |
| [`customer/pay-for-order-vnpay.puml`](customer/pay-for-order-vnpay.puml) | Thanh toán đơn hàng qua VNPAY | [`pay-for-order-vnpay.vi.md`](../usecase-specification/customer/pay-for-order-vnpay.vi.md) |
| [`customer/remove-cart-item.puml`](customer/remove-cart-item.puml) | Xóa sản phẩm khỏi giỏ | [`remove-cart-item.vi.md`](../usecase-specification/customer/remove-cart-item.vi.md) |
| [`customer/search-and-filter-products.puml`](customer/search-and-filter-products.puml) | Tìm kiếm và lọc sản phẩm | [`search-and-filter-products.vi.md`](../usecase-specification/customer/search-and-filter-products.vi.md) |
| [`customer/update-cart-item.puml`](customer/update-cart-item.puml) | Đổi số lượng trong giỏ | [`update-cart-item.vi.md`](../usecase-specification/customer/update-cart-item.vi.md) |
| [`customer/upload-image.puml`](customer/upload-image.puml) | Tải ảnh lên | [`upload-image.vi.md`](../usecase-specification/customer/upload-image.vi.md) |
| [`customer/use-ai-product-search.puml`](customer/use-ai-product-search.puml) | Tìm sản phẩm bằng trợ lý AI | [`use-ai-product-search.vi.md`](../usecase-specification/customer/use-ai-product-search.vi.md) |
| [`customer/view-ai-chat-history.puml`](customer/view-ai-chat-history.puml) | Xem lịch sử trò chuyện AI | [`view-ai-chat-history.vi.md`](../usecase-specification/customer/view-ai-chat-history.vi.md) |
| [`customer/view-cart.puml`](customer/view-cart.puml) | Xem giỏ hàng | [`view-cart.vi.md`](../usecase-specification/customer/view-cart.vi.md) |
| [`customer/view-my-orders.puml`](customer/view-my-orders.puml) | Xem đơn hàng của tôi | [`view-my-orders.vi.md`](../usecase-specification/customer/view-my-orders.vi.md) |
| [`customer/view-my-shipments.puml`](customer/view-my-shipments.puml) | Xem các đợt giao hàng của tôi | [`view-my-shipments.vi.md`](../usecase-specification/customer/view-my-shipments.vi.md) |
| [`customer/view-product-details.puml`](customer/view-product-details.puml) | Xem chi tiết sản phẩm | [`view-product-details.vi.md`](../usecase-specification/customer/view-product-details.vi.md) |
| [`customer/view-product-reviews.puml`](customer/view-product-reviews.puml) | Xem đánh giá sản phẩm | [`view-product-reviews.vi.md`](../usecase-specification/customer/view-product-reviews.vi.md) |
| [`customer/view-seller-shop.puml`](customer/view-seller-shop.puml) | Xem trang shop | [`view-seller-shop.vi.md`](../usecase-specification/customer/view-seller-shop.vi.md) |

### Seller

| Tệp | Use case | Spec |
|---|---|---|
| [`seller/cancel-order-item.puml`](seller/cancel-order-item.puml) | Hủy dòng hàng | [`cancel-order-item.vi.md`](../usecase-specification/seller/cancel-order-item.vi.md) |
| [`seller/chat-with-customer.puml`](seller/chat-with-customer.puml) | Trò chuyện với Customer | [`chat-with-customer.vi.md`](../usecase-specification/seller/chat-with-customer.vi.md) |
| [`seller/choose-delivery-method.puml`](seller/choose-delivery-method.puml) | Chọn cách giao hàng | [`choose-delivery-method.vi.md`](../usecase-specification/seller/choose-delivery-method.vi.md) |
| [`seller/confirm-order-item.puml`](seller/confirm-order-item.puml) | Xác nhận dòng hàng | [`confirm-order-item.vi.md`](../usecase-specification/seller/confirm-order-item.vi.md) |
| [`seller/create-product.puml`](seller/create-product.puml) | Tạo sản phẩm | [`create-product.vi.md`](../usecase-specification/seller/create-product.vi.md) |
| [`seller/create-shipment.puml`](seller/create-shipment.puml) | Tạo đợt giao | [`create-shipment.vi.md`](../usecase-specification/seller/create-shipment.vi.md) |
| [`seller/login.puml`](seller/login.puml) | Đăng nhập | [`login.vi.md`](../usecase-specification/seller/login.vi.md) |
| [`seller/manage-own-shipments.puml`](seller/manage-own-shipments.puml) | Quản lý đợt giao của tôi | [`manage-own-shipments.vi.md`](../usecase-specification/seller/manage-own-shipments.vi.md) |
| [`seller/manage-products.puml`](seller/manage-products.puml) | Quản lý sản phẩm | [`manage-products.vi.md`](../usecase-specification/seller/manage-products.vi.md) |
| [`seller/process-own-order-items.puml`](seller/process-own-order-items.puml) | Xử lý dòng hàng của tôi | [`process-own-order-items.vi.md`](../usecase-specification/seller/process-own-order-items.vi.md) |
| [`seller/unlist-product.puml`](seller/unlist-product.puml) | Ẩn sản phẩm | [`unlist-product.vi.md`](../usecase-specification/seller/unlist-product.vi.md) |
| [`seller/update-product.puml`](seller/update-product.puml) | Sửa sản phẩm | [`update-product.vi.md`](../usecase-specification/seller/update-product.vi.md) |
| [`seller/update-product-stock.puml`](seller/update-product-stock.puml) | Cập nhật tồn kho | [`update-product-stock.vi.md`](../usecase-specification/seller/update-product-stock.vi.md) |
| [`seller/update-shipment-status.puml`](seller/update-shipment-status.puml) | Cập nhật trạng thái đợt giao | [`update-shipment-status.vi.md`](../usecase-specification/seller/update-shipment-status.vi.md) |
| [`seller/upload-image.puml`](seller/upload-image.puml) | Tải ảnh lên | [`upload-image.vi.md`](../usecase-specification/seller/upload-image.vi.md) |
| [`seller/view-own-order-items.puml`](seller/view-own-order-items.puml) | Xem dòng hàng của tôi | [`view-own-order-items.vi.md`](../usecase-specification/seller/view-own-order-items.vi.md) |
| [`seller/view-own-products.puml`](seller/view-own-products.puml) | Xem sản phẩm của tôi | [`view-own-products.vi.md`](../usecase-specification/seller/view-own-products.vi.md) |
| [`seller/view-own-shipments.puml`](seller/view-own-shipments.puml) | Xem đợt giao của tôi | [`view-own-shipments.vi.md`](../usecase-specification/seller/view-own-shipments.vi.md) |

### Admin

| Tệp | Use case | Spec |
|---|---|---|
| [`admin/approve-or-suspend-seller.puml`](admin/approve-or-suspend-seller.puml) | Duyệt hoặc tạm ngưng Seller | [`approve-or-suspend-seller.vi.md`](../usecase-specification/admin/approve-or-suspend-seller.vi.md) |
| [`admin/create-category.puml`](admin/create-category.puml) | Tạo danh mục | [`create-category.vi.md`](../usecase-specification/admin/create-category.vi.md) |
| [`admin/create-user.puml`](admin/create-user.puml) | Tạo người dùng | [`create-user.vi.md`](../usecase-specification/admin/create-user.vi.md) |
| [`admin/create-voucher.puml`](admin/create-voucher.puml) | Tạo voucher | [`create-voucher.vi.md`](../usecase-specification/admin/create-voucher.vi.md) |
| [`admin/delete-category.puml`](admin/delete-category.puml) | Xóa danh mục | [`delete-category.vi.md`](../usecase-specification/admin/delete-category.vi.md) |
| [`admin/disable-user.puml`](admin/disable-user.puml) | Vô hiệu hóa người dùng | [`disable-user.vi.md`](../usecase-specification/admin/disable-user.vi.md) |
| [`admin/expire-voucher.puml`](admin/expire-voucher.puml) | Cho voucher hết hạn | [`expire-voucher.vi.md`](../usecase-specification/admin/expire-voucher.vi.md) |
| [`admin/login.puml`](admin/login.puml) | Đăng nhập | [`login.vi.md`](../usecase-specification/admin/login.vi.md) |
| [`admin/manage-carriers.puml`](admin/manage-carriers.puml) | Quản lý đơn vị vận chuyển | [`manage-carriers.vi.md`](../usecase-specification/admin/manage-carriers.vi.md) |
| [`admin/manage-categories.puml`](admin/manage-categories.puml) | Quản lý danh mục | [`manage-categories.vi.md`](../usecase-specification/admin/manage-categories.vi.md) |
| [`admin/manage-users.puml`](admin/manage-users.puml) | Quản lý người dùng | [`manage-users.vi.md`](../usecase-specification/admin/manage-users.vi.md) |
| [`admin/manage-vouchers.puml`](admin/manage-vouchers.puml) | Quản lý voucher | [`manage-vouchers.vi.md`](../usecase-specification/admin/manage-vouchers.vi.md) |
| [`admin/update-category.puml`](admin/update-category.puml) | Đổi tên danh mục | [`update-category.vi.md`](../usecase-specification/admin/update-category.vi.md) |
| [`admin/update-user.puml`](admin/update-user.puml) | Sửa người dùng | [`update-user.vi.md`](../usecase-specification/admin/update-user.vi.md) |
| [`admin/update-voucher.puml`](admin/update-voucher.puml) | Sửa voucher | [`update-voucher.vi.md`](../usecase-specification/admin/update-voucher.vi.md) |
| [`admin/view-categories.puml`](admin/view-categories.puml) | Xem danh mục | [`view-categories.vi.md`](../usecase-specification/admin/view-categories.vi.md) |
| [`admin/view-users.puml`](admin/view-users.puml) | Xem người dùng | [`view-users.vi.md`](../usecase-specification/admin/view-users.vi.md) |
| [`admin/view-vouchers.puml`](admin/view-vouchers.puml) | Xem voucher | [`view-vouchers.vi.md`](../usecase-specification/admin/view-vouchers.vi.md) |

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
