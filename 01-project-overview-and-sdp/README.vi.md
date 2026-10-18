# 01. Tổng quan Dự án & Kế hoạch Phát triển Phần mềm

## 1. Kiểm soát Tài liệu

| Trường | Giá trị |
|---|---|
| ID tài liệu | DOC-01 |
| Tiêu đề tài liệu | Tổng quan Dự án & Kế hoạch Phát triển Phần mềm |
| Dự án | E-Commerce Marketplace |
| Cơ sở nguồn | [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md) |
| Tài liệu tham khảo bổ trợ | [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml) |
| Trạng thái | Baseline lập kế hoạch |
| Kiến trúc cơ sở | Modular monolith |

## 2. Mục đích

Tài liệu này định nghĩa bối cảnh dự án, phạm vi sản phẩm, định hướng kỹ thuật, trình tự phát triển,
cổng chất lượng (quality gate) và tiêu chí hoàn thành cho E-Commerce Marketplace.

Tài liệu này chỉ giới hạn trong thông tin lập kế hoạch nằm trong `PLAN.md`. Tài liệu không khẳng định rằng một
tính năng được lập kế hoạch đã được triển khai.

## 3. Tổng quan Dự án

Sản phẩm là một marketplace thương mại điện tử được triển khai dưới dạng modular monolith:

- một ứng dụng Spring Boot;
- một tiến trình JVM;
- một đơn vị triển khai (deployment unit);
- một cơ sở dữ liệu PostgreSQL chính.

Client mobile là một ứng dụng React Native được xây dựng bằng Expo. Backend sở hữu phân quyền (authorization),
nghiệp vụ (business rule), transaction, callback thanh toán, quy tắc vận chuyển, chat giữa người với người (human chat) và tích hợp AI.

Marketplace hỗ trợ đơn hàng đa người bán (multi-seller order). Một đơn hàng có thể chứa các order item từ nhiều seller.
Shipment được tạo theo từng seller và có thể nhóm nhiều order item từ seller đó.

## 4. Phạm vi Sản phẩm

### 4.1 Năng lực của Customer

Customer có thể:

- duyệt (browse), tìm kiếm (search), lọc (filter) và phân trang (paginate) sản phẩm;
- quản lý cart item;
- đặt hàng (place order);
- thanh toán bằng COD hoặc VNPAY;
- theo dõi shipment;
- đánh giá (review) sản phẩm sau khi thỏa mãn các điều kiện mua hàng và giao hàng;
- chat với seller;
- sử dụng trợ lý AI của marketplace.

### 4.2 Năng lực của Seller

Seller có thể:

- quản lý sản phẩm và tồn kho (stock) của chính mình;
- xử lý order item của chính mình;
- lựa chọn platform delivery hoặc self delivery;
- tạo và quản lý shipment của chính mình;
- chat với customer.

### 4.3 Năng lực của Admin

Admin có thể:

- quản lý user, category và voucher;
- kiểm duyệt vi phạm;
- giám sát vận hành marketplace;
- cấu hình platform delivery;
- thực hiện các tác vụ quản trị như phê duyệt hoàn tiền (refund approval).

## 5. Ràng buộc Phát triển

Các quyết định sau đây là bắt buộc đối với hệ thống được lập kế hoạch:

1. Giữ hệ thống dưới dạng modular monolith. Không tách thành microservices.
2. Không đưa vào API Gateway, service discovery, Feign, REST nội bộ (internal REST), Kafka, Saga hoặc các mẫu
   transaction phân tán (distributed transaction pattern) trừ khi yêu cầu sau này yêu cầu rõ ràng.
3. Tổ chức backend theo feature, sau đó tách trách nhiệm kỹ thuật bên trong mỗi feature.
4. Giữ controller ở dạng thin. Nghiệp vụ thuộc về service hoặc domain logic.
5. Sử dụng PostgreSQL transaction cho checkout, cập nhật stock, tiêu thụ voucher và callback thanh toán.
6. Giữ Spring AI trong cùng monolith dưới feature `ai`.
7. Khởi đầu AI với các tool chỉ đọc (read-only tool). AI không được trực tiếp thay đổi trạng thái nghiệp vụ quan trọng (business-critical state).
8. React Native không được gọi trực tiếp LLM provider (DeepSeek). Các yêu cầu AI đi qua Spring Boot.

## 6. Cơ sở Công nghệ

Các version backend dưới đây phản ánh `backend/pom.xml`, `backend/Dockerfile` và stack Docker Compose
hiện tại. Frontend vẫn là khung sườn (scaffold), vì vậy version thư viện của nó cố ý chưa được chốt.

| Lĩnh vực | Công nghệ hoặc hướng tiếp cận | Trạng thái |
|---|---|---|
| Runtime backend | Spring Boot 3.3.5, Java 21, Maven 3.9+ | Current scaffold |
| API backend | Spring Web MVC, Validation, Springdoc OpenAPI 2.5.0, Actuator | Current scaffold |
| Runtime mobile | React Native with Expo | Scaffold; version TBD |
| Client HTTP | Axios | Planned frontend dependency |
| Trạng thái server (Server state) | TanStack Query | Planned frontend dependency |
| Lưu trữ token | Expo SecureStore | Planned frontend dependency |
| Cơ sở dữ liệu | PostgreSQL 16 with pgvector image | Current local container scaffold |
| Persistence | Spring Data JPA/Hibernate | Current backend dependency |
| Migration cơ sở dữ liệu | Flyway | Current backend dependency |
| Image container backend | `backend/Dockerfile` đa tầng (stage build Maven + runtime Temurin 21 JRE, user không phải root) | Current scaffold |
| Điều phối cục bộ | Docker Compose: PostgreSQL/pgvector + backend API, chờ DB healthy | Current scaffold |
| Xác thực | Spring Security with JJWT 0.12.6 | Current backend dependency; flow is phased |
| Chat giữa người với người | Spring WebSocket and mobile WebSocket client | Backend dependency; feature phased |
| AI | Spring AI with an external LLM provider | Planned backend phase |
| Nhà cung cấp thanh toán | VNPAY, with COD support | Planned feature |
| Kiểm thử | Spring Boot Test, JUnit 5, Testcontainers, H2 | Current backend scaffold |
| Biểu diễn tiền tệ | `BigDecimal` | Backend implementation rule |

## 7. Tổ chức Backend

Backend sử dụng package-by-feature. Các feature cấp cao nhất theo kế hoạch là:

```text
auth | user | catalog | cart | order | payment
shipment | voucher | review | chat | ai | common
```

Mỗi feature có thể chứa các layer kỹ thuật như `controller`, `service`, `repository`, `entity`,
`dto`, `mapper`, `security`, `config`, `websocket` hoặc `vnpay` khi cần thiết.

Dự án phải tránh một cấu trúc toàn cục duy nhất như `controller/`, `service/`, `repository/` và
`entity/`, vì cách đó phân tán một feature nghiệp vụ trên toàn bộ codebase.

## 8. Kế hoạch Phát triển Phần mềm

### Giai đoạn 0 - Nền tảng Dự án

- Khởi tạo Spring Boot monolith.
- Cấu hình PostgreSQL.
- Sử dụng Flyway cho các database migration có version.
- Tạo response lỗi chung (common error response).
- Thêm bộ xử lý ngoại lệ toàn cục (global exception handler).
- Cấu hình validation.
- Thiết lập quy ước package-by-feature.
- Thêm thiết lập test cơ sở (base test setup).

**Cổng hoàn thành:** ứng dụng khởi động được, database migration chạy thành công và `/actuator/health` ở trạng thái
healthy.

### Giai đoạn 1 - Auth và User

- Tạo user entity và repository.
- Thêm role.
- Triển khai register, login và refresh.
- Thêm JWT access token và refresh token.
- Mã hóa mật khẩu (password encoding).
- Cấu hình phân quyền (authorization) Spring Security.
- Thêm trạng thái phê duyệt seller nếu thuộc phạm vi MVP.

**Cổng hoàn thành:** việc login và phân quyền của CUSTOMER, SELLER và ADMIN có thể kiểm chứng được bằng test.

### Giai đoạn 2 - Catalog

- Thêm CRUD category cho Admin.
- Thêm CRUD product cho seller.
- Thêm quản lý stock.
- Thêm hành vi product active hoặc soft-delete.
- Thêm luồng duyệt (browse), tìm kiếm (search), lọc (filter) và phân trang (pagination) cho customer.

**Cổng hoàn thành:** quyền sở hữu của seller và luồng catalog của customer được bao phủ bởi integration test.

### Giai đoạn 3 - Cart

- Thêm tạo, cập nhật và xóa item.
- Giữ một product là duy nhất trong mỗi cart của customer.
- Validate product active và stock.
- Tính lại giá ở phía server (server-side).

**Cổng hoàn thành:** stock không hợp lệ và product không active không thể tiếp tục.

### Giai đoạn 4 - Checkout và Order

- Tạo order theo cơ chế transaction.
- Snapshot tên product và giá trong order item.
- Tách quyền sở hữu seller ở cấp độ order-item.
- Áp dụng voucher.
- Tính toán tổng tiền.
- Cập nhật stock.
- Đặt thời hạn prepayment là 24 giờ sau khi tạo order; COD tuân theo quy tắc thu tiền khi giao hàng (delivery collection rule).
- Chỉ hoàn thành COD sau khi tất cả các seller shipment bắt buộc, không bị hủy (non-cancelled) đều đã delivered trong MVP; không
  tự ý thêm thanh toán từng phần (partial settlement) cho các kết quả seller khác nhau.

**Cổng hoàn thành:** một cart có sản phẩm từ ít nhất hai seller tạo ra một order với các
order item được gán đúng.

### Giai đoạn 5 - Xử lý Order của Seller

- Truy vấn order item của seller.
- Confirm hoặc cancel order item.
- Thực thi quyền sở hữu (ownership).
- Ghi nhận lý do hủy (cancellation reason) nếu được yêu cầu.
- Không hoàn lại stock của shipment khi giao hàng chuyển thành `FAILED`.
- Chỉ hoàn lại số lượng đã giao (shipped quantity) sau khi shipment chuyển thành `RETURNED`.

### Giai đoạn 6 - Payment

- Tạo bản ghi payment COD.
- Tạo request VNPAY và xử lý callback.
- Validate chữ ký callback (callback signature).
- Đảm bảo callback có tính idempotent.
- Thử lại payment thất bại bằng cách tạo một bản ghi `Payment` khác.
- Cập nhật trạng thái order thành PAID theo quy tắc thanh toán.
- Hủy order prepayment sau 24 giờ và hoàn lại stock khi checkout đúng một lần; COD tuân theo
  quy tắc thu tiền khi giao hàng (delivery collection rule).

### Giai đoạn 7 - Shipment

- Cho phép seller chọn platform delivery hoặc self delivery.
- Chỉ tạo shipment từ các item đã confirm và hợp lệ.
- Thực thi một seller duy nhất trên tất cả các item trong một shipment.
- Theo dõi các chuyển đổi trạng thái shipment.
- Hỗ trợ `FAILED -> RETURNED`; chỉ hoàn lại stock sau `RETURNED`.
- Đặt order thành DELIVERED chỉ sau khi tất cả các shipment bắt buộc đã được delivered.

### Giai đoạn 8 - Quản trị Review và Voucher

- Kiểm tra điều kiện đánh giá (review eligibility) của customer.
- Thực thi một review cho mỗi customer/product.
- Thêm CRUD voucher cho Admin.
- Validate hiệu lực và mức sử dụng voucher.

### Giai đoạn 9 - Chat giữa người với người

- Ghép cặp customer và seller trong một conversation.
- Lưu trữ (persist) message.
- Thêm cập nhật thời gian thực (real-time) qua WebSocket.
- Tải lịch sử message ngoại tuyến (offline message history).
- Thêm cơ chế retry và xử lý idempotency cho message.

### Giai đoạn 10 - Spring AI

Triển khai AI theo thứ tự sau:

```text
ChatClient
 -> read-only tools
 -> chat memory
 -> RAG + pgvector
 -> semantic search/review summary
```

Không kích hoạt các write action của AI trong bản triển khai đầu tiên.

Phạm vi AI hiện tại là tìm kiếm sản phẩm cho unauthenticated/customer. Các tool order và
shipment cho authenticated customer là công việc read-only giai đoạn sau. Phân tích doanh thu của seller chỉ ở mức lập kế hoạch.

### Giai đoạn 11 - Tích hợp React Native

- Tích hợp auth và navigation.
- Tích hợp catalog, cart và checkout.
- Tích hợp order, payment và theo dõi shipment.
- Tích hợp review.
- Tích hợp human chat.
- Tích hợp UI chat AI.

### Giai đoạn 12 - Củng cố (Hardening)

- Thêm unit test và integration test.
- Thêm test quyền sở hữu bảo mật (security ownership test).
- Thêm test idempotency thanh toán (payment idempotency test).
- Thêm test order và shipment đa người bán (multi-seller).
- Thêm test phân quyền AI tool.
- Kiểm tra hiệu năng tìm kiếm sản phẩm và phân trang.
- Giữ stack Docker Compose (PostgreSQL/pgvector và image backend) cùng tài liệu triển khai luôn cập nhật.
- Tài liệu hóa triển khai production và triển khai test.

## 9. Cổng Chất lượng

### 9.1 Các kịch bản test tối thiểu

- Một đơn hàng multi-seller chứa sản phẩm từ Seller A và Seller B.
- Seller A không thể confirm order item của Seller B.
- Seller A không thể đưa item của Seller B vào shipment.
- Việc hủy shipment của Seller A không hủy sai shipment của Seller B.
- Seller A có thể ở trạng thái `DELIVERED` trong khi Seller B ở trạng thái `FAILED` hoặc `RETURNED` trong cùng một order.
- `FAILED` không hoàn lại stock; `RETURNED` chỉ hoàn lại số lượng đã giao của seller đó.
- Một order prepayment bị hủy tại thời điểm 24 giờ và hoàn lại stock khi checkout đúng một lần; COD được miễn.
- Một order COD có thể chuyển từ `PENDING` trực tiếp sang `DELIVERED`; payment của nó chuyển thành `COMPLETED` sau
  khi tất cả các shipment bắt buộc, không bị hủy (non-cancelled) đã được delivered.
- Một order giữ nguyên tên product và giá gốc sau khi product thay đổi.
- Một lần thử VNPAY thất bại giữ nguyên order.
- Một lần thử lại tạo ra bản ghi payment thứ hai.
- Một callback trùng lặp không làm trùng lặp việc hoàn thành payment.
- Voucher hết hạn và đã dùng hết bị từ chối.
- Việc sử dụng đồng thời voucher cuối cùng phải an toàn transaction (transaction-safe).
- Một customer không nhận được item không thể review item đó.
- Một customer đã nhận hàng (delivered) có thể review một lần.
- AI có thể truy cập order của customer hiện tại nhưng không truy cập order của customer khác.
- AI cho guest chỉ có thể truy cập tìm kiếm sản phẩm công khai và thông tin product.
- AI sử dụng RAG cho các câu hỏi chính sách (policy) và tool cho dữ liệu order/shipment trực tiếp (live).
- AI không thể cancel, refund, thay đổi stock hoặc thay đổi giá.

### 9.2 Định nghĩa Hoàn thành (Definition of Done)

Một tính năng chỉ hoàn thành khi:

- domain và business rule đã được triển khai;
- tồn tại DTO validation;
- đã thực thi phân quyền (authorization) và quyền sở hữu (ownership);
- các unit test và integration test liên quan đều pass;
- lỗi API nhất quán;
- tồn tại database migration;
- frontend có thể tiêu thụ API khi nó thuộc milestone hiện tại;
- các rule nghiệp vụ quan trọng (business-critical) không chỉ tồn tại ở frontend;
- các AI tool không bỏ qua phân quyền thông thường của ứng dụng.

## 10. Kiến trúc Mục tiêu Cấp Dự án

```text
React Native (Expo)
        |
        | REST / WebSocket
        v
+------------------------------------------------+
|           Spring Boot Modular Monolith         |
|                                                |
| auth | user | catalog | cart | order           |
| payment | shipment | voucher | review          |
| chat | ai | common                             |
|                                                |
| AI module                                      |
|   ChatClient                                   |
|   + Tool Calling -> existing feature services  |
|   + Chat Memory                                |
|   + RAG -> pgvector                            |
+--------------------------+---------------------+
                            |
                            v
                  PostgreSQL + pgvector
                            |
                            +--> external VNPAY
                            +--> external LLM provider
```

Mục tiêu vẫn là một monolith có thể triển khai được trong khi giữ các feature tách biệt nội bộ đủ để
duy trì khả năng bảo trì và thuận tiện cho phỏng vấn.
