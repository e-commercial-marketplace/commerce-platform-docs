# 03. Đặc tả Kiến trúc & Thiết kế

## 1. Kiểm soát Tài liệu

| Trường | Giá trị |
|---|---|
| ID tài liệu | DOC-03 |
| Tiêu đề tài liệu | Đặc tả Kiến trúc & Thiết kế |
| Dự án | E-Commerce Marketplace |
| Nguồn chính | [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md) |
| Tài liệu tham khảo mô hình class | [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml) |
| Trạng thái | Baseline kiến trúc và thiết kế |
| Kiểu kiến trúc | Modular monolith |

## 2. Mục đích

Tài liệu này mô tả kiến trúc mục tiêu, ranh giới module, tổ chức package, mô hình
miền, các luồng chính, thiết kế bảo mật, thiết kế persistence, tích hợp ngoài và chiến lược test.

Class diagram PlantUML là tài liệu tham khảo cho thuộc tính class, enumeration, quan hệ và
ràng buộc cấp class. Tài liệu này tóm tắt mô hình đó và không thay thế diagram nguồn.

## 3. Động lực Kiến trúc

Thiết kế chịu chi phối bởi các ràng buộc sau từ `PLAN.md`:

- hệ thống duy trì một ứng dụng Spring Boot, một tiến trình JVM, một đơn vị triển khai và một
  cơ sở dữ liệu PostgreSQL chính;
- các feature được tách theo năng lực nghiệp vụ, không theo một cây layer kỹ thuật toàn cục duy nhất;
- các lệnh gọi liên feature là các lệnh gọi phương thức Spring bean thông thường trong cùng một JVM;
- controller ở dạng thin và business rule được đặt trong service/domain logic;
- tính nhất quán transaction được yêu cầu cho checkout, stock, tiêu thụ voucher và callback
  thanh toán;
- các order prepayment hết hạn 24 giờ sau khi tạo và hoàn lại stock khi checkout đúng một lần; COD
  vẫn ở trạng thái pending cho đến khi tất cả các seller shipment bắt buộc đã delivered trong MVP;
- Spring AI nằm trong monolith và khởi đầu với các tool chỉ đọc (read-only tool);
- React Native không bao giờ gọi trực tiếp LLM provider.

## 4. Bối cảnh Hệ thống và Khung nhìn Triển khai

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

### 4.1 Các thành phần runtime

| Thành phần | Trách nhiệm |
|---|---|
| React Native (Expo) | Trình bày, điều hướng, trạng thái client, tiêu thụ REST/WebSocket |
| Spring Boot monolith | API, business rule, phân quyền, transaction, payment, shipment, chat và điều phối AI |
| PostgreSQL | Persistence chính của marketplace |
| pgvector | Lưu trữ embedding/vector cho RAG |
| VNPAY | Provider thanh toán trực tuyến ngoài |
| LLM provider | Provider model ngoài chỉ được truy cập bởi backend Spring AI |

### 4.2 Stack công nghệ

| Lớp | Công nghệ | Trạng thái |
|---|---|---|
| Runtime backend | Spring Boot 3.4.4, Java 21, Maven 3.9+ | Current scaffold |
| API backend | Spring Web MVC, Validation, Springdoc OpenAPI 2.7.0, Actuator | Current scaffold |
| Mapping backend | MapStruct 1.6.3 với Lombok binding để ánh xạ entity/DTO | Current backend dependency |
| Persistence backend | Spring Data JPA/Hibernate, PostgreSQL 16, Flyway | Current scaffold |
| Bảo mật backend | Spring Security and JJWT 0.12.6 | Current dependency; flow is phased |
| Realtime backend | Spring WebSocket | Dependency present; human chat is phased |
| Runtime mobile | React Native with Expo | Scaffold; version TBD |
| HTTP mobile | Axios | Planned frontend dependency |
| Trạng thái server mobile | TanStack Query | Planned frontend dependency |
| Lưu trữ bảo mật mobile | Expo SecureStore | Planned frontend dependency |
| AI | Spring AI, external LLM provider, pgvector RAG | Planned backend phase |
| Kiểm thử | Spring Boot Test, JUnit 5, Testcontainers, H2 | Current backend scaffold |

Frontend chưa có `package.json` hoặc dependency đã cài đặt. Các thư viện được liệt kê của nó là lựa chọn thiết kế
cho bản triển khai Expo đầu tiên, không phải bằng chứng của một bản build mobile đã hoàn thành.

## 5. Kiến trúc Module Logic

### 5.1 Các module feature

| Feature | Trách nhiệm chính | Các layer nội bộ theo kế hoạch |
|---|---|---|
| `auth` | Register, login, refresh, JWT, bảo mật (security) | `controller`, `service`, `repository`, `dto`, `security`, `config` |
| `user` | Dữ liệu user, role, trạng thái seller | `entity`, `repository`, `service`, `dto`, `mapper` |
| `catalog` | Category, product, stock, catalog của customer | `entity`, `repository`, `service`, `controller`, `dto`, `mapper` |
| `cart` | Cart item của customer và validate cart | `entity`, `repository`, `service`, `controller`, `dto` |
| `order` | Order, order item, checkout, xử lý order của seller | `entity`, `repository`, `service`, `controller`, `dto`, `mapper` |
| `payment` | Bản ghi payment, COD, tích hợp VNPAY | `entity`, `repository`, `service`, `controller`, `dto`, `vnpay` |
| `shipment` | Shipment của seller, phương thức giao hàng, tracking, trạng thái | `entity`, `repository`, `service`, `controller`, `dto` |
| `voucher` | Dữ liệu voucher, validate, sử dụng | `entity`, `repository`, `service`, `controller`, `dto` |
| `review` | Điều kiện review và review sản phẩm | `entity`, `repository`, `service`, `controller`, `dto` |
| `chat` | Conversation giữa người với người, message, WebSocket | `entity`, `repository`, `service`, `websocket`, `dto` |
| `ai` | ChatClient, tool, RAG, memory, persistence AI | `controller`, `service`, `repository`, `config`, `tools`, `rag`, `memory`, `entity`, `dto` |
| `common` | Lỗi chung, response, config, tiện ích | `exception`, `response`, `config`, `util` |

### 5.2 Cấu trúc package

```text
com.marketplace
├── auth
│   ├── controller
│   ├── service
│   ├── repository
│   ├── dto
│   ├── security
│   └── config
├── user
│   ├── entity
│   ├── repository
│   ├── service
│   ├── dto
│   └── mapper
├── catalog
├── cart
├── order
├── payment
├── shipment
├── voucher
├── review
├── chat
├── ai
└── common
```

Mỗi feature mở rộng thành các layer nêu trong mục 5.1 khi cần trách nhiệm đó. 
Kiến trúc tránh bố cục toàn cục `controller/`, `service/`, `repository/`, `entity/`.

### 5.3 Luồng gọi qua service và persistence

Luồng request đồng bộ theo kiến trúc mục tiêu:

```text
React Native -> FeatureController -> FeatureService -> FeatureRepository -> PostgreSQL
```

`FeatureService` là tầng service của feature: controller gọi trực tiếp trong cùng JVM; tầng này điều
phối use case và sở hữu business rules, có thể gọi feature khác thông qua service của feature đó.
Repository phụ trách persistence qua Spring Data JPA. Provider calls (ví dụ VNPAY hoặc LLM provider)
chỉ xuất hiện trong sequence diagram của feature tương ứng.

Entity chỉ chứa dữ liệu: field kèm mapping JPA, getter/setter và builder của Lombok, cùng callback
`@PrePersist`/`@PreUpdate` gán `createdAt`/`updatedAt`. Entity không có method tạo đối tượng, chuyển trạng
thái hay kiểm tra nghiệp vụ. Luật nghiệp vụ nằm ở service của feature sở hữu dữ liệu; feature khác cần
luật đó thì gọi service ấy.

Đây là kiến trúc mục tiêu, không khẳng định code đã hoàn thiện: backend scaffold hiện chưa có các
layer controller/service/repository.

## 6. Thiết kế Mô hình Miền

### 6.1 Các class từ diagram tham chiếu

Mô hình class chứa các entity sau:

```text
User, Category, Product, CartItem, Order, OrderItem,
Payment, Voucher, Shipment, Review, Conversation, Message,
AiChatSession, AiMessage
```

Mô hình class chứa các enumeration sau:

```text
Role, SellerStatus, OrderStatus, OrderItemStatus,
PaymentMethod, PaymentStatus, DeliveryMethod, ShipmentStatus,
AiRole
```

`AiRole` biểu diễn role của một message AI: `USER`, `ASSISTANT` hoặc `SYSTEM`.

### 6.2 Các quan hệ cốt lõi

| Quan hệ | Ý nghĩa thiết kế |
|---|---|
| `User -> CartItem -> Product` | Một customer sở hữu các cart item, mỗi item tham chiếu tới một product. |
| `User -> Order -> OrderItem` | Một customer đặt order; một order sở hữu một hoặc nhiều order item. |
| `OrderItem -> Product` | Mỗi order item tham chiếu tới product đã mua và lưu snapshot tên/giá product. |
| `OrderItem -> User` | Mỗi order item ghi nhận seller của nó. |
| `User -> Product` | Một seller sở hữu các product mà họ bán. |
| `Category -> Product` | Một category chứa các product. |
| `Order -> Payment` | Một order có thể có nhiều lần thử payment. |
| `Order -> Voucher` | Một order có thể áp dụng không hoặc một voucher. |
| `Order -> Shipment` | Một order có thể sinh nhiều shipment theo từng seller. |
| `Shipment -> OrderItem` | Một shipment chứa một hoặc nhiều order item từ cùng một seller. |
| `Shipment -> User` | Một shipment thuộc sở hữu/quản lý bởi một seller. |
| `User -> Review -> Product` | Một customer viết review cho product. |
| `Conversation -> Message` | Một conversation giữa người với người sở hữu các message của nó. |
| `Conversation -> Product/Order` | Một conversation giữa người với người có thể tham chiếu tới một product hoặc order tùy chọn. |
| `User -> AiChatSession -> AiMessage` | Persistence chat AI tách biệt với human chat. |

### 6.3 Các ràng buộc miền

Các ràng buộc sau là các bất biến thiết kế:

- `sellerStatus` chỉ bắt buộc khi `role = SELLER`.
- Một product có `price > 0` và `stock >= 0`.
- `active = false` nghĩa là product bị soft-delete hoặc unlisted.
- Tính duy nhất của cart là `(user, product)` và số lượng cart ít nhất là 1.
- Tính duy nhất của review là `(user, product)` và rating nằm trong `[1..5]`.
- Một seller chỉ có thể sửa các product, order item và shipment thuộc sở hữu.
- Tất cả các order item trong một shipment phải thuộc cùng một seller.
- `Shipment.seller == OrderItem.seller` đối với mọi item trong shipment đó.
- `Shipment.orderId == OrderItem.orderId` đối với mọi ánh xạ shipment.
- Một order item MVP được ánh xạ tới tối đa một shipment, sử dụng toàn bộ dòng order-item; các shipment từng phần,
  tách và thay thế bị hoãn lại.
- `transactionId` là bắt buộc và duy nhất đối với VNPAY.
- Một lần thử payment có thể thử lại bằng cách tạo một dòng payment khác.
- `trackingCode` và `shippedAt` là bắt buộc đối với `SHIPPED`.
- `deliveredAt` là bắt buộc đối với `DELIVERED`.
- `FAILED` là kết quả shipment ở phạm vi seller và không bao giờ tự hoàn lại stock.
- `RETURNED` nghĩa là kiện hàng đã đến kho của seller; chỉ khi đó số lượng của shipment đó mới được
  hoàn lại.
- `OrderStatus` vẫn là giá trị tổng hợp (aggregate) cấp order và không gộp các kết quả seller hỗn hợp.
- Việc hủy trực tiếp order-item được phép trước khi gán shipment. Một shipment `PENDING` hoặc
  `PREPARING` đã gán sử dụng đường hủy nguyên tử (atomic cancellation path) đánh dấu các dòng đã ánh xạ thành `CANCELLED` và
  giải phóng stock chưa giao đúng một lần; `SHIPPED` hoặc muộn hơn không thể hủy.
- Trước khi hoàn tất payment, transaction đó tính lại `Order.totalAmount` và `Payment.amount` đang pending
  từ các snapshot dòng còn lại. Các order bị hủy toàn bộ chuyển thành `CANCELLED`; nếu
  các dòng không bị hủy đã delivered là tất cả những gì còn lại, việc hoàn thành payment/order được đánh giá lại ngay lập tức.
- Tiền tệ được biểu diễn bằng `VND`, và MVP đầu tiên sử dụng phí vận chuyển (shipping fee) cố định `0.00`.
- Quy tắc human chat của MVP là duy nhất theo `(customer, seller)`.
- Chat AI tách biệt với human chat và chỉ đọc (read-only) đối với các thao tác Order, Payment và Shipment.

## 7. Thiết kế Trạng thái và Luồng công việc

### 7.1 Checkout của customer

```text
Cart
  -> validate active products
  -> validate stock
  -> validate voucher
  -> recalculate prices server-side
  -> create Order(PENDING)
  -> create OrderItems with name/price snapshots
  -> reserve or decrease stock
  -> set prepayment deadline to createdAt + 24 hours when the order requires prepayment
  -> save discount/shipping/total snapshots
  -> clear purchased cart items
```

Toàn bộ thao tác checkout nhạy cảm về tính nhất quán và nên đặt trong transaction.

### 7.2 Thực hiện đơn của seller

```text
OrderItem PENDING
      -> CONFIRMED
      -> included in Shipment

PENDING/CONFIRMED
      -> CANCELLED
```

Service phải so sánh seller đã xác thực với `orderItem.seller.id` trước khi thay đổi trạng thái.

### 7.3 Trạng thái shipment

```text
PENDING
  -> PREPARING
  -> SHIPPED
      -> DELIVERED
      -> FAILED
          -> RETURNED

PENDING/PREPARING
  -> CANCELLED
```

`SHIPPED` yêu cầu `trackingCode` và `shippedAt`. `DELIVERED` yêu cầu `deliveredAt`. `FAILED` không
hoàn lại stock. Sau khi kiện hàng đến kho của seller, shipment chuyển sang `RETURNED`
và chỉ quá trình chuyển đổi đó mới hoàn lại số lượng đã giao của nó. Order được đặt thành `DELIVERED` chỉ sau khi
tất cả các shipment bắt buộc, không bị hủy (non-cancelled) đã delivered; các kết quả seller hỗn hợp vẫn hiển thị theo từng shipment.
Việc hủy một shipment `PENDING`/`PREPARING` đã gán giải phóng stock chưa giao và không phải là thao tác
`RETURNED`.
Thao tác này cũng tính lại số tiền phải trả hiện tại mà không thay đổi các snapshot dòng lịch sử.
Đối với các kết quả cuối hỗn hợp, order vẫn là một container tổng hợp (aggregate container); API hiển thị kết quả theo từng
shipment và không thêm một enum lỗi cấp order mới.

### 7.4 Trạng thái payment

COD:

```text
Payment(method=COD, status=PENDING)
  -> COMPLETED when all required non-cancelled shipments are DELIVERED
```

Quá trình chuyển đổi `DELIVERED` bắt buộc cuối cùng là tác nhân thu tiền COD (COD collection trigger) được xử lý bởi các service shipment/payment
trong một transaction. MVP không ghi nhận thanh toán từng phần COD hoặc hoàn tiền cho các kết quả
`FAILED`/`RETURNED` hỗn hợp của seller.

VNPAY:

```text
Create Payment(PENDING)
 -> redirect/create VNPAY request
 -> callback
      COMPLETED
      or FAILED
```

Callback phải validate chữ ký/checksum và sử dụng định danh transaction/tham chiếu duy nhất để
xử lý idempotent.

## 8. Thiết kế API và Tích hợp

### 8.1 Ranh giới REST

Các nhóm REST theo kế hoạch là:

```text
/api/v1/auth
/api/v1/products
/api/v1/categories
/api/v1/seller/products
/api/v1/cart
/api/v1/orders
/api/v1/seller/order-items
/api/v1/vouchers
/api/v1/payments/vnpay
/api/v1/seller/shipments
/api/v1/seller/carriers
/api/v1/admin/carriers
/api/v1/products/{productId}/reviews
/api/v1/conversations
/api/v1/ai
```

Controller thể hiện các mối quan tâm vận chuyển (transport concern). Service thực thi business rule, quyền sở hữu, ranh giới transaction
và các lệnh gọi tới các feature service khác.

Prefix `/api/v1` được thêm tập trung cho `@RestController` của ứng dụng (`api.prefix` trong
`application.yml`, áp dụng bởi `WebConfig`) nên controller khai báo route không kèm prefix. Actuator,
OpenAPI/Swagger UI và `/error` giữ đường dẫn không version, còn security matcher trong
`SecurityConfig` dùng URL đã có prefix.

### 8.2 Ranh giới chat giữa người với người

Human chat sử dụng `WS /ws/chat` cho cập nhật thời gian thực và các endpoint REST cho lịch sử conversation/message.
Persistence message và retry/idempotency nằm trong feature `chat`.

### 8.3 Ranh giới AI

```text
AiController
    -> AiService
        -> ChatClient
            -> read-only tools
            -> RAG
            -> chat memory
```

Các AI tool gọi các application service hiện có. Chúng xác định authenticated customer từ Spring
Security context cho dữ liệu được bảo vệ và không chấp nhận các giá trị `userId` được bảo vệ tùy ý từ
model hoặc client. Quyền truy cập AI của guest giới hạn ở tìm kiếm sản phẩm công khai.

## 9. Thiết kế AI

### 9.1 Trình tự triển khai

```text
ChatClient
 -> read-only tool calling
 -> AiChatSession/AiMessage memory
 -> RAG + pgvector
 -> semantic product search/review summary
```

### 9.2 Các tool chỉ đọc

| Nhóm tool | Năng lực theo kế hoạch |
|---|---|
| `ProductTools` | `searchProducts(ProductSearchQuery)`, `getProduct` cho dữ liệu catalog công khai |
| `OrderTools` | `getMyOrders`, `getMyOrder(orderId)` cho customer sau này |
| `ShipmentTools` | `getShipmentStatus(orderId)`, `getTrackingInformation(orderId)` cho customer sau này |
| `VoucherTools` | `checkVoucher(code)` cho customer sau này |

Ranh giới tìm kiếm sản phẩm là:

```text
Natural-language request
  -> constrained ProductSearchQuery
  -> validation of fields, sort, and page limits
  -> catalog service search
  -> grounded natural-language answer
```

Phân tích doanh thu của seller chỉ ở mức lập kế hoạch.

Lớp AI ban đầu không được cung cấp `cancelOrder`, `refundPayment`, `changePrice`, `updateStock`,
`markDelivered` hoặc `deleteReview`.

### 9.3 Ranh giới RAG

RAG dành cho nội dung bán tĩnh như chính sách đổi trả (return policy), chính sách vận chuyển (shipping policy), hướng dẫn thanh toán (payment guide), chính sách voucher,
quy tắc seller và FAQ.

Trạng thái order trực tiếp (live), trạng thái shipment, stock, tình trạng voucher và giá product sử dụng tool calling
thay vì RAG.

## 10. Thiết kế Bảo mật

- Sử dụng Spring Security với luồng JWT access và refresh.
- Băm mật khẩu bằng `PasswordEncoder`; không bao giờ lưu plaintext.
- Thực thi kiểm tra role qua phân quyền endpoint/service.
- Thực thi kiểm tra quyền sở hữu (ownership) bổ sung cho role.
- Ngăn seller quản lý product, order item hoặc shipment của seller khác.
- Giới hạn truy vấn order/shipment của customer trong authenticated customer.
- Validate mọi chữ ký/checksum của callback VNPAY.
- Đảm bảo callback thanh toán có tính idempotent.
- Xác định danh tính AI tool được bảo vệ từ Spring Security context; chỉ cho phép guest trên các thao tác
  tìm kiếm sản phẩm công khai.
- Không bao giờ tin tưởng role hoặc user ID do React Native cung cấp.
- Giữ API key của provider trên backend.
- Trong production, tắt việc bao gồm message lỗi framework và lỗi binding; trả về response
  `INTERNAL_ERROR` an toàn cho các lỗi không mong đợi.

## 11. Thiết kế Persistence và Migration

Sử dụng Flyway cho các thay đổi schema có thứ tự. Thứ tự migration theo kế hoạch là:

```text
V1  users
V2-V11 placeholder lịch sử (không sửa sau khi Flyway đã ghi nhận)
V12 refresh_tokens
V13 các bảng domain: categories, products, cart_items, orders/items, vouchers, payments,
    shipments/mapping, reviews, conversations/messages, ai_chat_sessions/messages
V14 password_reset_codes (bổ sung Phase 1)
V15 cart_items.created_at (sắp xếp giỏ hàng ở Phase 3)
V16+ pgvector extension và AI document/vector tables (hoãn đến khi hoàn tất thiết kế RAG schema)
```

Các ràng buộc và index quan trọng bao gồm:

- duy nhất `user.email`;
- duy nhất `category.name`;
- duy nhất `(cart_item.user_id, cart_item.product_id)`;
- transaction/tham chiếu VNPAY duy nhất khi áp dụng được;
- duy nhất `(review.user_id, review.product_id)`;
- các index trên `product.seller_id`, `order.status`, `order_item.seller_id`, `shipment.seller_id` và
  `shipment.status`.
- index trên `order.created_at` cho quá trình quét hết hạn order prepayment 24 giờ.

Giá trị tiền tệ dùng `BigDecimal`.

## 12. Ranh giới Thiết kế Frontend

Cấu trúc React Native hướng theo feature:

```text
src
├── features
│   ├── auth
│   ├── catalog
│   ├── cart
│   ├── checkout
│   ├── orders
│   ├── shipment
│   ├── review
│   ├── chat
│   └── ai
├── components
├── navigation
├── lib/api
├── hooks
├── store
└── types
```

React Native sở hữu trình bày và trạng thái client. Spring Boot vẫn là nơi có thẩm quyền đối với phân quyền,
ngay cả khi UI mobile ẩn một thao tác.

## 13. Thiết kế Kiểm thử

### 13.1 Thiết kế Kiểm thử Backend (Kế hoạch)

Kiến trúc backend về sau phải được kiểm chứng bằng:

- unit test;
- integration test;
- test quyền sở hữu bảo mật (security ownership test);
- test idempotency thanh toán (payment idempotency test);
- test order/shipment đa người bán (multi-seller);
- test phân quyền AI tool;
- kiểm tra hiệu năng tìm kiếm sản phẩm và phân trang.

Các test liên module quan trọng nhất bao phủ cách ly seller (seller isolation), snapshot giá order, thử lại payment,
xử lý callback trùng lặp, đồng thời voucher, điều kiện review và cách ly dữ liệu AI.

Tất cả nội dung trong tiểu mục này là **Kế hoạch**. Chúng mô tả ý định thiết kế và chưa được triển
khai. Chưa có integration, idempotency, ownership hay performance test nào của backend tồn tại trong
cột mốc hiện tại.

### 13.2 Kiểm thử Frontend (Đã triển khai)

Client React Native / Expo hiện có **45 automated test frontend** trong 8 suite, phân loại như sau:

| Nhóm | Số lượng |
|---|---|
| Unit Test | 18 |
| Component Test | 13 |
| Hook/Context Test | 12 |
| Screen/Rendering Test | 2 |
| **Tổng** | **45** |

Stack gồm Jest `~29.7.0` với preset `jest-expo` `~57.0.6` và `@testing-library/react-native` `12.8.1`.

Kết quả đã kiểm chứng cho frontend commit `36e02a6` (merge qua `866897b`): 8 suite pass, 45 test
pass, 0 failed, 0 skipped, 0 snapshots.

Đây chỉ là unit, component, hook/context và screen-rendering test giới hạn. Chúng không kiểm chứng
tích hợp backend thật, hành vi cart phía server, checkout, orders, payment, shipment, reviews,
voucher, chat, AI hay notifications.

Đặc tả test case đầy đủ, coverage và truy vết:
[`04-testing/README.vi.md`](../04-testing/README.vi.md).

## 14. Quyết định Kiến trúc và Các loại trừ

Thiết kế cố ý loại trừ các nội dung sau cho đến khi một yêu cầu sau này chứng minh chúng:

- microservices;
- API Gateway;
- service discovery;
- Feign hoặc REST liên service;
- Kafka;
- Saga;
- transaction phân tán (distributed transaction);
- write action AI ban đầu.

Các loại trừ này giữ một monolith có thể triển khai duy nhất và giữ các nghiệp vụ bên trong ranh giới
transaction và phân quyền Spring thông thường.
