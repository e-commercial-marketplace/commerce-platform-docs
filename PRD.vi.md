# Tài liệu yêu cầu sản phẩm (PRD)

## 1. Định nghĩa sản phẩm

E-Commerce Marketplace là multi-seller marketplace được delivery dưới dạng modular monolith. Customers
có thể browse và purchase products từ nhiều sellers trong một order. Sellers quản lý catalog,
stock, order items và shipments của riêng mình. Admins quản lý platform-level operations.

Hệ thống mục tiêu gồm React Native (Expo) mobile client và Spring Boot backend kết nối
tới PostgreSQL và pgvector. Payment dùng COD và VNPAY. AI assistance được cung cấp qua Spring AI
bên trong backend.

### 1.1 Phạm vi MVP

Mobile MVP đầu tiên là customer-facing shopping flow. Các screen references được cung cấp định nghĩa
capabilities cần implement, không phải visual specification pixel-perfect.

#### Các mốc bàn giao

- **MVP tối thiểu (bắt buộc đạt):** luồng mua sắm của customer dưới đây. Các màn hình Seller/Admin
  không bắt buộc trong mobile MVP đầu tiên, nhưng backend vẫn phải enforce các domain rule của chúng.
- **Full roadmap:** app 3 role (`CUSTOMER`, `SELLER`, `ADMIN`) với quản lý catalog/stock/order/
  shipment của seller, quản lý user/category/voucher của admin, human chat, tích hợp VNPAY và
  lát cắt AI đầy đủ.

#### Trong phạm vi

- Public catalog browsing, search, category filtering, price/rating filtering, pagination và trình bày grid/list.
- Product detail với product information, rating summary, quantity selection và các action add-to-cart/buy-now.
- Customer registration/login bắt buộc trước server-backed cart checkout.
- Đăng ký public mặc định tạo tài khoản `CUSTOMER`. Onboarding seller được cung cấp trên màn hình tạo
  tài khoản: đơn đăng ký seller tạo tài khoản `SELLER` với trạng thái `PENDING`. Đăng ký public
  không bao giờ được tạo tài khoản `ADMIN`.
- Đơn đăng ký seller mới kích hoạt email thông báo; admin duyệt sẽ kích hoạt email xác nhận thành công
  gửi cho seller. Gửi email là best-effort và không được rollback đăng ký/duyệt khi gửi thất bại.
- Tài khoản admin đầu tiên được bootstrap từ cấu hình môi trường khi chưa có admin.
  Credentials không bao giờ hard-code trong repo.
- Thêm, cập nhật, xóa cart item và server-side total calculation.
- Checkout với shipping-address snapshot và COD payment. VNPAY vẫn là payment slice tiếp theo.
- Tạo order, payment result và tra cứu order cơ bản của customer.
- Product reviews chỉ gồm rating và comment.
- Public/customer AI product search và hỏi đáp theo context marketplace qua search flow hiện có; guest
  searches có thể stateless và không yêu cầu đăng nhập. AI cho guest chỉ giới hạn ở dữ liệu công khai
  (sản phẩm, FAQ, chính sách); câu hỏi về đơn hàng/vận chuyển/tài khoản yêu cầu customer đã xác thực.

#### Hoãn khỏi mobile MVP đầu tiên

- Seller và Admin screens.
- Human chat, Favorites và full Account screens.
- Advanced shipment-tracking UI; shipment state transitions vẫn là backend responsibilities.
- Voucher administration và voucher application UI.
- Product variants như color/size, review tags và review image uploads.
- Tích hợp VNPAY request/callback.
- AI order/shipment tools, RAG, persisted AI chat history và seller revenue analysis.

Backend vẫn phải enforce các domain rules không được thể hiện bằng dedicated MVP screen:
multi-seller order ownership, 24-hour prepayment cancellation, independent seller shipments,
xử lý `FAILED`/`RETURNED` và khôi phục stock chỉ sau `RETURNED`.

## 2. Actors sản phẩm

| Actor | Product responsibilities |
|---|---|
| UNAUTHENTICATED USER | Browse public catalog data và dùng AI cho public product search |
| CUSTOMER | Browse products, quản lý cart, đặt orders, pay, theo dõi shipments, review products, chat và dùng AI assistance |
| SELLER | Quản lý owned products và stock, xử lý owned order items, chọn delivery method, quản lý owned shipments và chat với customers |
| ADMIN | Quản lý users, duyệt seller, categories, vouchers và đơn vị vận chuyển |

## 3. Năng lực sản phẩm

### Catalog và cart

- Customers browse, search, filter và paginate products.
- Sellers quản lý products và stock của riêng mình.
- Customers thêm, cập nhật và xóa cart items.
- Một customer cart chứa tối đa một row cho mỗi product.
- Product activity, stock và server-side price được backend validate.
- MVP đầu tiên dùng `VND` và fixed `0.00` shipping fee; backend tính final total.

### Multi-seller checkout và orders

- Một order có thể chứa products từ nhiều sellers.
- Backend validate active products, stock, vouchers và prices.
- Product name và price được snapshot trong `OrderItem` tại checkout.
- Stock, totals, voucher use và cart clearing là một phần của một consistency-sensitive operation.
- Một order yêu cầu prepayment bị cancelled 24 giờ sau khi tạo nếu payment chưa hoàn tất;
  stock reserved/decreased cho order đó được khôi phục đúng một lần. COD orders tuân theo delivery
  collection rule và được miễn prepayment deadline này.
- Một order item chỉ có thể direct-cancelled trước shipment assignment. Một assigned `PENDING` hoặc
  `PREPARING` shipment có thể cancelled atomically, thao tác này cancels các mapped lines của nó và releases
  unshipped stock đúng một lần; cancellation sau `SHIPPED` bị rejected.
- Trước payment completion, một valid cancellation tính lại order payable total và pending
  payment amount từ các remaining line snapshots. Nếu mọi lines đều cancelled, order
  `CANCELLED` và không COD nào được collected; nếu chỉ còn lại delivered non-cancelled lines, thì
  order/payment được finalized trong cùng transaction.
- Sellers chỉ xử lý order items của riêng mình.

### Payment

- COD bắt đầu như pending payment và chỉ completes sau khi mọi required seller shipments đã delivered
  trong MVP. Partial COD settlement cho mixed seller outcomes được hoãn lại.
- VNPAY bắt đầu như pending payment và kết thúc qua validated callback.
- Một failed VNPAY attempt giữ order nguyên vẹn; retry tạo payment record khác.
- Callback processing có tính idempotent.

### Delivery và shipment

- Sellers chọn platform delivery hoặc self delivery.
- Một shipment chứa items từ đúng một seller.
- Một multi-seller order có thể tạo nhiều shipments.
- Shipment state tách biệt khỏi order-item processing state.
- Mỗi seller shipment được báo cáo độc lập: `DELIVERED`, `FAILED` hoặc `RETURNED`.
- `FAILED` không khôi phục stock. Sau khi package tới seller warehouse, shipment được
  marked `RETURNED`, và chỉ lúc đó shipped quantity của seller đó mới được restored.
- Một shipment thuộc về đúng một order và một seller. Trong MVP, một order item được gán cho tối
  đa một shipment, và shipment mapping dùng whole order-item lines; split và replacement shipments
  được hoãn lại.
- Một order chỉ becomes delivered sau khi mọi required shipments đã delivered.
- Mixed seller outcomes được hiển thị từ shipment list; chúng không bị collapsed thành false
  order-level `DELIVERED` result hay order-level failure enum mới.

### Reviews và vouchers

- Chỉ customer đã purchased và received product mới được review nó.
- Một customer chỉ được review một product một lần.
- Một order dùng tối đa một voucher.
- Expired, exhausted hoặc invalid vouchers đều bị rejected.

### Human chat và AI assistant

- Human customer/seller conversations persist messages và hỗ trợ real-time WebSocket updates.
- Human chat và AI chat là các khái niệm tách biệt.
- Unauthenticated users có thể dùng AI cho public product search và product information.
- `CUSTOMER` có thể dùng product search và sau đó là read-only tools cho orders, shipments,
  tracking và vouchers của riêng mình.
- `SELLER` và `ADMIN` ngoài AI scope hiện tại. Seller revenue analysis chỉ ở mức planning.
- AI ánh xạ natural-language requests thành constrained product-search query, để catalog service
  thực hiện authoritative search và sinh answers từ các returned products.
- AI dùng RAG cho semi-static policies và tools cho live transactional data.
- AI ban đầu không cancel orders, refund payments, đổi prices, cập nhật stock, mark delivery hay
  delete reviews.

## 4. Quy tắc sản phẩm

- Sản phẩm vẫn là single deployable modular monolith.
- Business-critical authorization được enforce bởi Spring Boot, không chỉ bởi mobile UI.
- Role checks và ownership checks đều bắt buộc.
- React Native không bao giờ nhận hay dùng LLM provider API key.
- Money dùng `BigDecimal`.
- Consistency-sensitive operations dùng PostgreSQL transactions.

## 5. Ranh giới sản phẩm

Các mục sau không thuộc initial target trừ khi requirement sau này bổ sung tường minh:

- microservices;
- API Gateway hoặc service discovery;
- Feign hoặc internal inter-service REST;
- Kafka, Saga hoặc distributed transactions;
- AI write operations mutate trực tiếp business-critical state.

## 6. Ranh giới nghiệm thu sản phẩm

Sản phẩm được coi là ready cho feature milestone chỉ khi domain rules, DTO validation,
authorization và ownership, consistent API errors, migration, tests liên quan và frontend consumption
đã hoàn tất ở những nơi feature thuộc frontend milestone hiện tại.

Các acceptance scenarios quan trọng gồm multi-seller isolation, independent shipment outcomes,
failed-payment retry, 24-hour prepayment-order cancellation và stock restoration, duplicate callback
handling, voucher concurrency, review eligibility và guest/customer AI data boundaries.
