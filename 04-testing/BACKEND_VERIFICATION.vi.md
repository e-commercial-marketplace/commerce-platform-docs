# Kiểm chứng backend và truy vết yêu cầu

## Baseline và kết quả

- Backend HEAD `2f34fef`; kiểm tra ngày 2026-10-28 trên Windows, Java 21.0.8. Trạng thái working
  tree tại thời điểm chạy không được ghi nhận đầy đủ; không khẳng định chạy trên commit sạch.
- Lệnh tại thư mục `backend`: `mvn test` — **BUILD SUCCESS, 231 test, 0 failures,
  0 errors, 0 skipped**. Kết quả chỉ áp dụng cho lần chạy và revision này.
- Maven Surefire tạo báo cáo tại `backend/target/surefire-reports/` trên máy chạy; thư mục
  này là output build, không phải bằng chứng đã được lưu trong repo docs. Không có coverage đo ở lần này.
- Test profile dùng H2 in-memory, Flyway tắt; nhiều API test dùng MockMvc. Các test có thể dùng mock;
  không coi kết quả này là chứng nhận PostgreSQL, Redis, VNPAY hay provider AI thật.
- Postman E2E trong `backend/e2e/README.md` mô tả hành trình HTTP, PostgreSQL và Flyway ở local;
  **chưa chạy trong lần kiểm chứng này**. Collection tạo dữ liệu thật, chỉ chạy local; không đổi
  base URL sang Render. VNPAY sandbox, email thật và job nền không thuộc collection đó.

## Ma trận truy vết và đặc tả test chọn lọc

Các ID lấy từ [SRS](../02-requirements-specification-srs/README.vi.md). Đây là ma trận **một phần**,
không phải nghiệm thu toàn bộ yêu cầu. Tên class/method nằm trong `backend/src/test/java/com/marketplace/`.
Các dòng dưới đã chạy thành công trong lần `mvn test` trên; kết quả là ở cấp test cụ thể, không có
nghĩa toàn bộ yêu cầu hoặc frontend đã đạt. SRS và contract vẫn là nguồn đặc tả chính.

| TC | Yêu cầu | Điều kiện và bước kiểm thử | Kết quả mong đợi | Test backend |
|---|---|---|---|---|
| BE-ORD-01 | FR-ORD-01, FR-ORD-08 | Cart có dòng của hai seller; checkout các dòng được chọn. | Tạo một order chứa các dòng đã chọn. | `controller/OrderApiTest#checkoutOfTwoSellersCreatesOneOrderFromTheSelectedLinesOnly` |
| BE-ORD-02 | FR-CART-04, FR-ORD-04 | Một dòng không đủ kho tại checkout. | 409; không tạo order hoặc trừ kho dòng hợp lệ. | `controller/OrderApiTest#insufficientStockRejectsTheWholeCheckout` |
| BE-ORD-03 | FR-ORD-09 | Customer truy cập order của customer khác. | 404, không lộ order. | `controller/OrderApiTest#anotherCustomersOrderIs404` |
| BE-PAY-01 | FR-PAY-05 | Gửi IPN thành công rồi lặp lại. | Order đã thanh toán; callback lặp không thay đổi kết quả. | `controller/PaymentApiTest#aSuccessfulIpnMarksTheOrderPaidAndARepeatChangesNothing` |
| BE-SHP-01 | FR-ORD-13 | Customer xác nhận đã nhận order DELIVERED. | Order chuyển COMPLETED. | `controller/ShipmentApiTest#theCustomerClosesADeliveredOrderByConfirmingReceipt` |
| BE-VCH-01 | FR-VCH-05 | Checkout với voucher hợp lệ. | Áp dụng giảm giá và tiêu thụ một lượt. | `controller/VoucherApiTest#checkoutAppliesThePercentageAndConsumesOneUse` |
| BE-AI-01 | FR-AI-10 | Guest gửi câu hỏi AI. | Có câu trả lời, không trả session ID và số session không tăng. Test này không kiểm tra trực tiếp số tin nhắn. | `controller/AiApiTest#aGuestGetsAnAiAnswerAndNothingIsStored` |

Khoảng trống ưu tiên: chạy Postman E2E local và lưu kết quả; kiểm thử VNPAY sandbox thật,
email thật, Flyway/PostgreSQL, job nền và tải; kiểm thử đồng thời stock/voucher trên PostgreSQL.
Test `theLastItemCanBeSoldOnlyOnce` chạy lần lượt, không chứng minh cạnh tranh đồng thời.
Truy vết các FR/NFR còn lại, frontend integration và UAT **chưa hoàn thành trong ma trận này**.

## Trạng thái phạm vi theo backend baseline

Theo `backend/README.md`, backend có auth/user, catalog, cart/order, COD/VNPAY sandbox, shipment,
voucher/review, AI và Redis cache tùy chọn. Human chat, RAG và Google login chưa triển khai.
Backend có test H2 đã chạy như trên; E2E lần này chưa chạy. Trạng thái mobile và triển khai thực tế
chưa được kiểm tra trong lần này; xem [snapshot frontend](README.vi.md) và xác minh riêng trước khi
bàn giao. Use case và sơ đồ của chức năng chưa triển khai là thiết kế dự kiến, không phải bằng chứng đã chạy.
