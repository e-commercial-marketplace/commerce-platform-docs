# 02. Đặc tả Yêu cầu (Requirements Specification - SRS)

## 1. Kiểm soát Tài liệu

| Trường | Giá trị |
|---|---|
| ID tài liệu | DOC-02 |
| Tiêu đề tài liệu | Đặc tả Yêu cầu (Requirements Specification - SRS) |
| Dự án | E-Commerce Marketplace |
| Nguồn chính | [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md) |
| Tài liệu tham khảo mô hình miền | [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml) |
| Trạng thái | Baseline yêu cầu |

## 2. Mục đích và Phạm vi

SRS này định nghĩa các yêu cầu chức năng, business rule, giao diện ngoài, yêu cầu bảo mật,
yêu cầu dữ liệu và kịch bản kiểm chứng theo kế hoạch cho E-Commerce Marketplace.

Các yêu cầu dưới đây chỉ giới hạn trong các năng lực và ràng buộc đã nêu trong `PLAN.md` và
class diagram tham chiếu. Các tham chiếu màn hình định nghĩa năng lực MVP chứ không phải yêu cầu
hình ảnh chi tiết đến từng pixel. Chi tiết request/response được định nghĩa trong [`API_CONTRACT.vi.md`](../API_CONTRACT.vi.md).

### 2.1 Định hướng MVP mobile đầu tiên

Cột mốc frontend đầu tiên triển khai hành trình mua sắm của customer. Các domain rule của backend vẫn có
hiệu lực ngay cả khi màn hình seller/admin hoặc màn hình hậu mãi tương ứng bị hoãn lại.

| Lĩnh vực | Quyết định cho MVP đầu tiên | Quyết định hoãn lại |
|---|---|---|
| Catalog | Duyệt công khai, tìm kiếm, bộ lọc category/price/rating, phân trang, chi tiết sản phẩm | Các màn hình catalog của Seller/Admin |
| Mô hình product | Một ảnh sản phẩm duy nhất và không có biến thể màu sắc/kích thước | `ProductVariant` và tồn kho biến thể |
| Xác thực | Register/login/refresh của Customer trước checkout được server hỗ trợ | Phê duyệt seller và Admin UI |
| Cart | Thêm, cập nhật, xóa và tính tổng ở phía server (server-side totals) | Đồng bộ cart guest chỉ ở phía client |
| Checkout | Snapshot địa chỉ giao hàng, tiền tệ `VND` cố định, phí vận chuyển `0.00` và tạo order theo transaction | Quản lý sổ địa chỉ và giá vận chuyển động |
| Payment | Ưu tiên COD; payment vẫn ở trạng thái pending cho đến khi tất cả các seller shipment bắt buộc đã delivered | Tích hợp request/callback VNPAY và thanh toán từng phần COD |
| Voucher | Không có UI voucher hoặc áp dụng khi checkout trong lát cắt đầu tiên | Validate voucher và tiêu thụ an toàn đồng thời |
| Review | Một review hợp lệ của customer gồm rating và comment | Thẻ review và tải ảnh lên |
| AI | Tìm kiếm sản phẩm công khai/customer qua `POST /api/v1/ai/ask`; request của guest có thể stateless | Các tool order/shipment, RAG, chat AI có lưu trữ, phân tích doanh thu seller |
| Giao hàng | Trạng thái shipment và quy tắc stock của backend là bắt buộc | UI theo dõi của Customer và màn hình shipment của Seller |
| Chat giữa người với người | Không thuộc cột mốc mobile đầu tiên | Các conversation qua WebSocket |

Hình dạng API của MVP và hợp đồng lỗi chung (common error contract) được định nghĩa trong [`API_CONTRACT.vi.md`](../API_CONTRACT.vi.md).

## 3. Quan điểm Sản phẩm

Hệ thống là một modular monolith gồm một client React Native (Expo) và một backend Spring Boot.
Backend giao tiếp với PostgreSQL/pgvector, VNPAY và một LLM provider ngoài.

Backend là nơi có thẩm quyền đối với phân quyền (authorization), quyền sở hữu (ownership), business rule, transaction, callback
thanh toán và quyền truy cập AI tool.

## 4. Tác nhân và Nhóm Người dùng

| Tác nhân | Trách nhiệm |
|---|---|
| UNAUTHENTICATED USER | Duyệt dữ liệu catalog công khai và sử dụng AI chỉ cho tìm kiếm sản phẩm công khai |
| CUSTOMER | Duyệt sản phẩm, quản lý cart, đặt hàng, thanh toán, theo dõi shipment, đánh giá sản phẩm, chat, sử dụng hỗ trợ AI |
| SELLER | Quản lý product và stock thuộc sở hữu, xử lý order item thuộc sở hữu, lựa chọn giao hàng, quản lý shipment thuộc sở hữu, chat |
| ADMIN | Quản lý user/category/voucher, kiểm duyệt vi phạm, giám sát vận hành, cấu hình platform delivery, phê duyệt các tác vụ quản trị như hoàn tiền |
| Payment provider | Trả về dữ liệu callback thanh toán VNPAY |
| LLM provider | Cung cấp response của model thông qua module AI của backend |

## 5. Yêu cầu Chức năng

### 5.1 Xác thực và quản lý user

| ID | Yêu cầu |
|---|---|
| FR-AUTH-01 | Hệ thống phải hỗ trợ các role customer, seller và admin. |
| FR-AUTH-02 | Hệ thống phải hỗ trợ các thao tác register, login và refresh. |
| FR-AUTH-03 | Hệ thống phải sử dụng băm mật khẩu (password hashing) qua Spring Security `PasswordEncoder`; không được lưu mật khẩu dạng plaintext. |
| FR-AUTH-04 | Hệ thống phải sử dụng luồng JWT access và refresh. |
| FR-AUTH-05 | Hệ thống phải áp dụng các trạng thái phê duyệt seller: `PENDING`, `APPROVED`, `REJECTED` và `SUSPENDED`. Chỉ seller `APPROVED` mới được quản lý sản phẩm. |
| FR-AUTH-06 | Hệ thống phải cho admin xem danh sách, xem chi tiết, tạo, cập nhật và vô hiệu hóa người dùng, lọc danh sách theo role và từ khóa. Vô hiệu hóa tài khoản vẫn giữ dữ liệu và chặn đăng nhập, refresh. |
| FR-AUTH-07 | Hệ thống phải cho admin duyệt, tạm khóa hoặc từ chối seller. Admin không được tự vô hiệu hóa hoặc tự hạ role của chính mình. |
| FR-AUTH-08 | Hệ thống phải cho người dùng đã đăng nhập đổi mật khẩu bằng cách nhập mật khẩu hiện tại. Đổi thành công phải thu hồi mọi refresh token của người dùng và trả về cặp token mới. |
| FR-AUTH-09 | Hệ thống phải cho người dùng quên mật khẩu yêu cầu một mã 6 chữ số dùng một lần qua email và đặt mật khẩu mới bằng mã đó. Yêu cầu không được để lộ email đã đăng ký hay chưa. Mã hết hạn sau 15 phút, cho sai tối đa 5 lần và chỉ dùng được một lần. Đặt lại thành công phải thu hồi mọi refresh token của người dùng. |
| FR-AUTH-10 | Hệ thống phải gửi email qua nhà cung cấp cấu hình bằng biến môi trường. Lỗi gửi email không được để lộ cho client biết tài khoản có tồn tại hay không. |

FR-AUTH-08 đến FR-AUTH-10 là phần bổ sung vào Phase 1.

Các endpoint theo kế hoạch:

```text
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh

GET    /api/v1/users/me
GET    /api/v1/admin/users
POST   /api/v1/admin/users
GET    /api/v1/admin/users/{id}
PATCH  /api/v1/admin/users/{id}
DELETE /api/v1/admin/users/{id}
PATCH  /api/v1/admin/sellers/{id}/approve
PATCH  /api/v1/admin/sellers/{id}/suspend

POST   /api/v1/auth/password/forgot
POST   /api/v1/auth/password/reset
PATCH  /api/v1/users/me/password
```

### 5.2 Catalog

| ID | Yêu cầu |
|---|---|
| FR-CAT-01 | Customer phải có thể duyệt sản phẩm. |
| FR-CAT-02 | Customer phải có thể tìm kiếm, lọc và phân trang sản phẩm. |
| FR-CAT-03 | Customer phải có thể xem một product và các category. |
| FR-CAT-04 | Seller phải có thể tạo, cập nhật, soft-delete/unlist và cập nhật stock cho product của chính mình. |
| FR-CAT-05 | Một product active phải thuộc về đúng một category và một seller. |
| FR-CAT-06 | Một seller không được sửa product thuộc sở hữu của seller khác. |
| FR-CAT-07 | Customer phải có thể xem shop của một seller (tên shop là tên của seller) và các product đang bán của seller đó. |
| FR-CAT-08 | Seller phải có thể liệt kê và xem product của chính mình, kể cả product đã unlist. |

Các endpoint theo kế hoạch:

```text
GET    /api/v1/products
GET    /api/v1/products/{id}
GET    /api/v1/categories
GET    /api/v1/sellers/{id}

GET    /api/v1/seller/products
GET    /api/v1/seller/products/{id}
POST   /api/v1/seller/products
PATCH  /api/v1/seller/products/{id}
DELETE /api/v1/seller/products/{id}
PATCH  /api/v1/seller/products/{id}/stock
```

### 5.3 Cart

| ID | Yêu cầu |
|---|---|
| FR-CART-01 | Một customer phải có thể xem cart. |
| FR-CART-02 | Một customer phải có thể thêm, cập nhật và xóa cart item. |
| FR-CART-03 | Một cart của customer chứa tối đa một dòng cho mỗi product. |
| FR-CART-04 | Hệ thống phải validate rằng product đang active và đủ stock trước khi checkout. |
| FR-CART-05 | Backend phải tính lại giá ở phía server (server-side). |

Các endpoint theo kế hoạch:

```text
GET    /api/v1/cart
POST   /api/v1/cart/items
PATCH  /api/v1/cart/items/{id}
DELETE /api/v1/cart/items/{id}
```

### 5.4 Checkout và order

| ID | Yêu cầu |
|---|---|
| FR-ORD-01 | Hệ thống phải tạo order theo cơ chế transaction từ các cart item hợp lệ. |
| FR-ORD-02 | Một order phải chứa ít nhất một order item. |
| FR-ORD-03 | Hệ thống phải snapshot tên product và đơn giá trong mỗi order item tại thời điểm checkout. |
| FR-ORD-04 | Hệ thống phải giữ chỗ (reserve) hoặc giảm stock trong quá trình checkout. |
| FR-ORD-05 | Hệ thống phải validate và áp dụng voucher theo quy tắc hiệu lực và sử dụng của nó. |
| FR-ORD-06 | Hệ thống phải tính toán và lưu snapshot discount, shipping và total. |
| FR-ORD-07 | Hệ thống phải xóa các cart item đã mua sau khi checkout thành công. |
| FR-ORD-08 | Một order có thể chứa các order item từ nhiều seller. |
| FR-ORD-09 | Một customer phải có thể liệt kê và chỉ xem các order của chính mình. |
| FR-ORD-10 | Hệ thống phải hủy order prepayment 24 giờ sau `Order.createdAt` khi payment chưa hoàn tất và hoàn lại stock khi checkout của nó đúng một lần; các order COD không có thời hạn prepayment. |
| FR-ORD-11 | Việc hủy trực tiếp order-item chỉ được phép trước khi gán shipment. Một shipment `PENDING`/`PREPARING` đã gán có thể bị hủy nguyên tử (atomically), đánh dấu các dòng đã ánh xạ thành `CANCELLED` và giải phóng stock chưa giao đúng một lần; việc hủy sau `SHIPPED` hoặc muộn hơn bị từ chối. |
| FR-ORD-12 | Trước khi hoàn tất payment, một lần hủy hợp lệ phải tính lại tổng phải trả của order và mọi số tiền payment đang pending từ các snapshot order-item còn lại; nếu tất cả các dòng đều bị hủy, order phải chuyển thành `CANCELLED`. |

Các endpoint theo kế hoạch:

```text
POST /api/v1/orders
GET  /api/v1/orders
GET  /api/v1/orders/{id}
```

### 5.5 Xử lý order của seller

| ID | Yêu cầu |
|---|---|
| FR-FUL-01 | Một seller phải có thể truy vấn các order item của chính mình. |
| FR-FUL-02 | Một seller phải có thể confirm hoặc cancel các order item của chính mình. |
| FR-FUL-03 | Các thao tác của seller phải thực thi `orderItem.seller.id == authenticatedSeller.id`. |
| FR-FUL-04 | Việc hủy của seller không được ảnh hưởng đến order item hoặc shipment của seller khác. |

Các endpoint theo kế hoạch:

```text
GET   /api/v1/seller/order-items
PATCH /api/v1/seller/order-items/{id}/confirm
PATCH /api/v1/seller/order-items/{id}/cancel
```

### 5.6 Payment

| ID | Yêu cầu |
|---|---|
| FR-PAY-01 | Hệ thống phải hỗ trợ COD và VNPAY. |
| FR-PAY-02 | Một payment COD phải bắt đầu ở trạng thái `PENDING` và chỉ hoàn thành sau khi tất cả các seller shipment bắt buộc, không bị hủy (non-cancelled) đều ở trạng thái `DELIVERED`; thanh toán từng phần (partial settlement) cho các kết quả `FAILED`/`RETURNED` hỗn hợp bị hoãn lại. |
| FR-PAY-03 | Một payment VNPAY phải được tạo ở trạng thái `PENDING`, gửi tới provider và hoàn thành hoặc thất bại bởi callback. |
| FR-PAY-04 | Hệ thống phải validate chữ ký/checksum của callback VNPAY. |
| FR-PAY-05 | Các callback thanh toán phải có tính idempotent. |
| FR-PAY-06 | Một lần thử VNPAY thất bại không được phá hủy order. Một lần thử lại phải tạo một bản ghi `Payment` khác. |
| FR-PAY-07 | Các định danh transaction/tham chiếu VNPAY phải là duy nhất khi áp dụng được. |

Các endpoint theo kế hoạch:

```text
POST /api/v1/payments/vnpay/create
POST /api/v1/payments/vnpay/callback
```

### 5.7 Shipment và giao hàng

| ID | Yêu cầu |
|---|---|
| FR-SHIP-01 | Một seller phải chọn `PLATFORM_DELIVERY` hoặc `SELF_DELIVERY`. |
| FR-SHIP-02 | Một shipment phải chứa các order item từ đúng một seller. |
| FR-SHIP-03 | Một shipment có thể nhóm nhiều order item từ cùng một seller. |
| FR-SHIP-04 | Một shipment chỉ được tạo từ các order item đã confirm, hợp lệ, thuộc cùng một order và seller. |
| FR-SHIP-05 | Khi trạng thái chuyển thành `SHIPPED`, `trackingCode` và `shippedAt` phải tồn tại. |
| FR-SHIP-06 | Khi trạng thái chuyển thành `DELIVERED`, `deliveredAt` phải tồn tại. |
| FR-SHIP-07 | Order phải chuyển thành `DELIVERED` chỉ sau khi tất cả các shipment bắt buộc đã được delivered. |
| FR-SHIP-08 | Một customer phải có thể truy vấn thông tin shipment chỉ cho order của chính mình. |
| FR-SHIP-09 | Mỗi seller shipment phải báo cáo kết quả `DELIVERED`, `FAILED` hoặc `RETURNED` của riêng mình. |
| FR-SHIP-10 | Một order đa người bán (multi-seller) với các kết quả shipment hỗn hợp không được báo cáo thành `DELIVERED` ở cấp order. |
| FR-SHIP-11 | Một shipment bị đánh dấu `FAILED` không được hoàn lại stock. |
| FR-SHIP-12 | Một shipment chỉ có thể bị đánh dấu `RETURNED` sau khi kiện hàng đến kho của seller; chỉ khi đó số lượng đã giao của nó mới được hoàn lại. |
| FR-SHIP-13 | Trong MVP, mỗi order item có thể thuộc về tối đa một shipment và ánh xạ shipment sử dụng toàn bộ dòng order-item; các shipment từng phần dòng (partial-line), tách (split) và thay thế (replacement) bị hoãn lại. |

Các endpoint theo kế hoạch:

```text
POST  /api/v1/seller/shipments
GET   /api/v1/seller/shipments
PATCH /api/v1/seller/shipments/{id}/status
GET   /api/v1/orders/{orderId}/shipments
```

### 5.8 Voucher

| ID | Yêu cầu |
|---|---|
| FR-VCH-01 | Một order sử dụng tối đa một voucher. |
| FR-VCH-02 | Hệ thống phải validate hiệu lực và mức sử dụng voucher. |
| FR-VCH-03 | Hệ thống phải từ chối voucher hết hạn hoặc đã dùng hết. |
| FR-VCH-04 | Việc tiêu thụ voucher phải sử dụng update điều kiện nguyên tử (atomic conditional update) hoặc chiến lược optimistic-locking tương đương để các request đồng thời không vượt quá `usageLimit`. |

Các endpoint theo kế hoạch:

```text
POST /api/v1/orders/{id}/voucher
GET  /api/v1/vouchers/{code}/validate
```

### 5.9 Review

| ID | Yêu cầu |
|---|---|
| FR-REV-01 | Chỉ customer đã mua và nhận product mới đủ điều kiện review nó. |
| FR-REV-02 | Một customer có tối đa một review cho mỗi product. |
| FR-REV-03 | Rating của review phải nằm trong khoảng 1 đến 5. |

Các endpoint theo kế hoạch:

```text
POST /api/v1/products/{productId}/reviews
GET  /api/v1/products/{productId}/reviews
```

### 5.10 Chat giữa người với người

| ID | Yêu cầu |
|---|---|
| FR-CHAT-01 | Hệ thống phải hỗ trợ các conversation customer/seller. |
| FR-CHAT-02 | Hệ thống phải lưu trữ message và hỗ trợ tải lịch sử ngoại tuyến (offline history loading). |
| FR-CHAT-03 | Hệ thống phải cung cấp cập nhật thời gian thực qua WebSocket. |
| FR-CHAT-04 | Hệ thống phải cung cấp chiến lược retry/idempotency cho message. |
| FR-CHAT-05 | Chat giữa người với người phải tách biệt với chat AI. |

Các giao diện theo kế hoạch:

```text
WS  /ws/chat
GET /api/v1/conversations
GET /api/v1/conversations/{id}/messages
```

### 5.11 Trợ lý AI

| ID | Yêu cầu |
|---|---|
| FR-AI-01 | Trợ lý AI phải chạy bên trong Spring Boot monolith thông qua Spring AI. |
| FR-AI-02 | Hệ thống phải cung cấp `POST /api/v1/ai/ask`. |
| FR-AI-03 | Bản triển khai AI đầu tiên phải sử dụng các tool chỉ đọc (read-only tool). |
| FR-AI-04 | Các AI tool được bảo vệ phải xác định authenticated customer từ Spring Security context; request của guest chỉ giới hạn ở dữ liệu catalog công khai. |
| FR-AI-05 | Các AI tool phải gọi các application service hiện có thay vì bỏ qua chúng bằng truy cập repository trực tiếp, trừ khi có lý do chính đáng. |
| FR-AI-06 | Năng lực AI ban đầu phải ánh xạ request ngôn ngữ tự nhiên thành một truy vấn tìm kiếm sản phẩm bị giới hạn (constrained product-search query), gọi catalog service và sinh câu trả lời dựa trên các product trả về. |
| FR-AI-07 | Các giai đoạn authenticated-customer sau này có thể cung cấp các thao tác read-only cho order hiện tại, shipment, tracking và voucher. |
| FR-AI-08 | Hệ thống không được cung cấp các AI tool ban đầu để hủy order, hoàn tiền payment, thay đổi giá, cập nhật stock, đánh dấu giao hàng hoặc xóa review. |
| FR-AI-09 | RAG phải được dùng cho các chính sách và FAQ bán tĩnh; các application service phải được dùng cho dữ liệu catalog và customer trực tiếp (live). |
| FR-AI-10 | Các session và message AI đã lưu phải thuộc về authenticated customer; các request tìm kiếm sản phẩm của guest có thể stateless. |
| FR-AI-11 | Phân tích doanh thu của seller chỉ ở mức lập kế hoạch và không được kích hoạt trong phạm vi AI hiện tại. |

Các endpoint theo kế hoạch:

```text
POST /api/v1/ai/ask                         (public catalog search; CUSTOMER protected scope)
GET  /api/v1/ai/sessions                    (CUSTOMER only)
GET  /api/v1/ai/sessions/{id}/messages      (CUSTOMER and own session only)
```

## 6. Quy tắc Nghiệp vụ và Bất biến Miền

Các quy tắc sau áp dụng trên toàn bộ feature:

- Một product thuộc về đúng một category và một seller.
- Một seller chỉ có thể sửa các product thuộc sở hữu của mình.
- Seller của shipment phải bằng seller của mọi order item trong shipment đó.
- Một shipment chứa các item từ đúng một seller.
- `shipment.orderId == orderItem.orderId` đối với mọi ánh xạ shipment.
- MVP sử dụng tối đa một shipment cho mỗi seller trong một order và chỉ dùng toàn bộ dòng order-item.
- `OrderItemStatus` theo dõi xử lý của seller; tiến độ vận chuyển thuộc về `ShipmentStatus`.
- `OrderStatus` là giá trị tổng hợp (aggregate) cấp order; `FAILED` và `RETURNED` là các giá trị `ShipmentStatus` ở phạm vi seller.
- Một order có thể có nhiều lần thử payment.
- Tiền tệ được biểu diễn dưới dạng `VND`; MVP đầu tiên sử dụng phí vận chuyển (shipping fee) cố định `0.00`.
- Một order sử dụng tối đa một voucher.
- Chat giữa người với người và chat AI là các khái niệm riêng biệt.
- AI không được thực hiện các tác vụ nghiệp vụ không thể đảo ngược trong bản triển khai ban đầu.

Class diagram tham chiếu còn định nghĩa thêm các ràng buộc mô hình sau:

- `sellerStatus` chỉ bắt buộc khi `role = SELLER`.
- Product có `price > 0`, `stock >= 0`, và `active = false` nghĩa là soft-deleted hoặc unlisted.
- `transactionId` là bắt buộc và duy nhất đối với VNPAY.
- Tính duy nhất của cart là `(user, product)` và `quantity >= 1`.
- Tính duy nhất của review là `(user, product)` và rating nằm trong `[1..5]`.
- `trackingCode` và `shippedAt` là bắt buộc khi shipment ở trạng thái `SHIPPED`.
- `deliveredAt` là bắt buộc khi shipment ở trạng thái `DELIVERED`.
- `sellerStatus` nullable trừ khi `role = SELLER`; các trường payment, shipment, chat và AI tùy chọn
  vẫn nullable cho đến khi trạng thái của chúng yêu cầu một giá trị.
- Quy tắc human chat của MVP là duy nhất theo `(customer, seller)`.

## 7. Yêu cầu Phi chức năng

| ID | Yêu cầu |
|---|---|
| NFR-ARCH-01 | Backend phải duy trì một modular monolith và một đơn vị có thể triển khai. |
| NFR-ARCH-02 | Các feature phải giao tiếp qua các lệnh gọi Spring bean thông thường bên trong JVM. |
| NFR-SEC-01 | Luồng JWT access/refresh phải được dùng cho xác thực. |
| NFR-SEC-02 | Cả kiểm tra role và kiểm tra quyền sở hữu (ownership) đều phải được thực thi. |
| NFR-SEC-03 | Hệ thống không được tin tưởng role hoặc user ID do React Native cung cấp. |
| NFR-SEC-04 | Các API key của provider không được lộ ra React Native. |
| NFR-DATA-01 | Tiền tệ phải dùng `BigDecimal`. |
| NFR-DATA-02 | Các luồng nhạy cảm về tính nhất quán phải dùng PostgreSQL transaction. |
| NFR-DATA-03 | Các thay đổi cơ sở dữ liệu phải được quản lý bởi Flyway. |
| NFR-API-01 | Lỗi API phải sử dụng hợp đồng response nhất quán. |
| NFR-API-02 | Tìm kiếm sản phẩm và phân trang phải được kiểm tra hiệu năng trong giai đoạn củng cố (hardening). |
| NFR-API-03 | Response production không được lộ chi tiết ngoại lệ nội bộ hoặc message binding của framework; các lỗi không mong đợi phải dùng `INTERNAL_ERROR`. |
| NFR-AI-01 | Các AI tool được bảo vệ phải xác định authenticated customer và tuân thủ phân quyền thông thường của ứng dụng; quyền truy cập của guest chỉ giới hạn ở catalog công khai. |
| NFR-AI-02 | RAG không được dùng làm nguồn sự thật (source of truth) cho dữ liệu transaction trực tiếp (live). |

## 8. Yêu cầu Giao diện Ngoài

### 8.1 Client mobile

React Native phải sử dụng backend cho các nghiệp vụ. Lớp dữ liệu client theo kế hoạch có thể dùng
Axios cho HTTP, TanStack Query cho trạng thái server (server state), SecureStore cho token và một client WebSocket/STOMP hoặc
client WebSocket đã chọn cho human chat.

Frontend có thể ẩn các thao tác không được phép truy cập, nhưng việc phân quyền vẫn phải do Spring Boot thực thi.

### 8.2 Nhà cung cấp thanh toán

Tích hợp VNPAY phải hỗ trợ tạo request, chuyển hướng (redirect), xử lý callback, validate chữ ký/checksum,
định danh transaction/tham chiếu duy nhất và hoàn thành có tính idempotent.

### 8.3 LLM provider

DeepSeek provider đã chọn (API tương thích OpenAI) phải được cấu hình trên backend. Client mobile
không bao giờ được gọi trực tiếp provider.

### 8.4 Cơ sở dữ liệu và vector store

PostgreSQL lưu trữ dữ liệu marketplace. pgvector lưu trữ embedding cho giai đoạn RAG. Dữ liệu transaction trực tiếp
(live) phải được đọc qua các application service/tool thay vì được coi là nội dung RAG tĩnh.

### 8.5 Nhà cung cấp email

Backend phải gửi email giao dịch, ví dụ mã đặt lại mật khẩu, qua SMTP cấu hình bằng biến môi trường.
Khi phát triển dùng Gmail SMTP với App Password; chưa chọn nhà cung cấp cho production. Khi tắt email lúc phát triển local, mã được ghi vào log của ứng dụng thay vì
gửi đi; production phải bật email.

## 9. Kiểm chứng và Nghiệm thu

Mỗi feature chỉ được nghiệm thu khi business rule, DTO validation, phân quyền/quyền sở hữu,
lỗi nhất quán, database migration và các test liên quan của nó đã hoàn tất. Việc frontend tiêu thụ cũng được
yêu cầu khi feature thuộc cột mốc frontend hiện tại.

Các kịch bản kiểm chứng tối thiểu được liệt kê trong `PLAN.md` và được tóm tắt trong mục 9.1 của
[`01-project-overview-and-sdp/README.vi.md`](../01-project-overview-and-sdp/README.vi.md).
