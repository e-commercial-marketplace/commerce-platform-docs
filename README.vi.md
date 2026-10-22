# Hướng dẫn tài liệu

Thư mục này chứa tài liệu planning, requirements, architecture, data model, system overview và
tài liệu cấp feature cho E-Commerce Marketplace.

Tài liệu dựa trên:

- [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md), là
  implementation và scope baseline;
- [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml), là
  domain class-model reference.

Các sơ đồ Mermaid trong thư mục này là documentation view dẫn xuất từ các nguồn trên. Chúng
không thay thế database migration hay Java implementation code.

## Thứ tự đọc khuyến nghị

### 1. Project và kế hoạch delivery

Đọc [`01-project-overview-and-sdp/README.vi.md`](01-project-overview-and-sdp/README.vi.md) trước. Phần này
giải thích project scope, roles, technology baseline, development phases, quality gates và Definition
of Done.

### 2. Requirements

Đọc [`02-requirements-specification-srs/README.vi.md`](02-requirements-specification-srs/README.vi.md) để
hiểu functional requirements, non-functional requirements, business rules, API boundaries,
security và acceptance scenarios.

### 3. Architecture và thiết kế

Đọc [`03-architecture-and-design-specification/README.vi.md`](03-architecture-and-design-specification/README.vi.md)
về cấu trúc modular-monolith, feature boundaries, workflows, AI design, persistence plan và
security design.

### 4. Tài liệu tham chiếu gốc

| Document | Purpose |
|---|---|
| [`PRD.vi.md`](PRD.vi.md) | Product scope, actors, capabilities, business goals và product acceptance boundary |
| [`API_CONTRACT.vi.md`](API_CONTRACT.vi.md) | MVP request/response DTOs, query parameters, errors, authentication và state rules |
| [`SYSTEM_OVERVIEW.vi.md`](SYSTEM_OVERVIEW.vi.md) | Runtime components và giao tiếp giữa mobile, backend, database, VNPAY và LLM provider |
| [`ERD.vi.md`](ERD.vi.md) | Logical data relationships và persistence constraints |
| [`CLASS_DIAGRAM.vi.md`](CLASS_DIAGRAM.vi.md) | Full domain class diagram dạng Mermaid, dựa trên PlantUML reference |

### 5. Tài liệu feature

Sau khi đọc các tài liệu gốc, mở feature mà bạn đang implement:

| Feature | Documentation |
|---|---|
| Auth | [`features/auth/README.vi.md`](features/auth/README.vi.md) |
| User | [`features/user/README.vi.md`](features/user/README.vi.md) |
| Catalog | [`features/catalog/README.vi.md`](features/catalog/README.vi.md) |
| Cart | [`features/cart/README.vi.md`](features/cart/README.vi.md) |
| Order | [`features/order/README.vi.md`](features/order/README.vi.md) |
| Payment | [`features/payment/README.vi.md`](features/payment/README.vi.md) |
| Shipment | [`features/shipment/README.vi.md`](features/shipment/README.vi.md) |
| Voucher | [`features/voucher/README.vi.md`](features/voucher/README.vi.md) |
| Review | [`features/review/README.vi.md`](features/review/README.vi.md) |
| Human chat | [`features/chat/README.vi.md`](features/chat/README.vi.md) |
| AI | [`features/ai/README.vi.md`](features/ai/README.vi.md) |
| Common | [`features/common/README.vi.md`](features/common/README.vi.md) |

Mỗi feature README giải thích responsibility, actors được phép, ownership rules, planned API area,
table relationships, field meanings và focused Mermaid class diagram. Dùng focused diagram và
mục data-model khi implement một feature để không phải luôn giữ toàn bộ domain
diagram trong tầm nhìn.

### 6. Sơ đồ use-case

Mở [`usecase-diagram/README.vi.md`](usecase-diagram/README.vi.md) để xem các sơ đồ draw.io có thể chỉnh sửa được phân tách
theo `UNAUTHENTICATED USER`, `CUSTOMER`, `SELLER` và `ADMIN`. Dùng các sơ đồ này để kiểm tra actor
permissions và các quan hệ `<<include>> Login` và `<<extend>>` bắt buộc trước khi implement
một endpoint hoặc screen.

### 7. Sơ đồ hoạt động

Mở [`activity-diagram/README.vi.md`](activity-diagram/README.vi.md) để xem mỗi use case trong
`usecase-specification/` một sơ đồ hoạt động PlantUML, phân theo tác nhân. Dùng các sơ đồ này để đi
từng bước của luồng — gồm validation, luồng thay thế và luồng ngoại lệ — trước khi implement hoặc
review một endpoint hay screen.

### 8. Sơ đồ tuần tự

Mở [`sequence-diagram/README.vi.md`](sequence-diagram/README.vi.md) để xem sequence diagram kỹ thuật
cho từng use case. Các sơ đồ Mermaid mô tả request đi qua controller của feature, service, repository
và database; provider call chỉ được vẽ khi contract có căn cứ.
Đây là kiến trúc mục tiêu, không khẳng định backend scaffold hiện tại đã có đủ các layer đó.

### 9. Kiểm thử

Đọc [`04-testing/README.vi.md`](04-testing/README.vi.md) về bộ automated test của frontend: chiến
lược kiểm thử, đặc tả test case đầy đủ, kết quả đã kiểm chứng, coverage và truy vết requirement. Tài
liệu này chỉ bao gồm các automated test frontend đã triển khai; integration, system và acceptance
test của backend vẫn ở trạng thái kế hoạch.

## Ánh xạ feature frontend-backend

React Native scaffold tách user journeys trong khi backend nhóm các business rules liên quan:

| Frontend feature | Backend documentation boundary |
|---|---|
| `checkout` | `order` + `payment` |
| `orders` | `order` + `shipment` + payment state display |
| `catalog` | `catalog` |
| `ai` | `ai` + public `catalog` search |

Ánh xạ này giúp tránh nhầm lẫn các thư mục frontend thành các backend bounded
contexts bổ sung.

## Cách dùng tài liệu khi coding

1. Xác nhận feature có thuộc mobile MVP đầu tiên không trong `PRD.vi.md` và SRS.
2. Xác nhận request/response shape trong [`API_CONTRACT.vi.md`](API_CONTRACT.vi.md).
3. Kiểm tra actor và ownership rules trước khi thêm endpoint hoặc service method.
4. Kiểm tra `ERD.vi.md` trước khi thêm hoặc thay đổi persistence relationships.
5. Kiểm tra `CLASS_DIAGRAM.vi.md` và feature sub-diagram trước khi thêm domain class hoặc field.
6. Kiểm tra `SYSTEM_OVERVIEW.vi.md` trước khi đưa vào external call hoặc cross-feature dependency mới.
7. Cập nhật feature README liên quan khi thiết kế đã thống nhất thay đổi.
8. Với multi-seller delivery, đọc từng `ShipmentStatus`; không suy diễn seller outcomes từ
   order-level `OrderStatus`.

## Ghi chú đặt tên

Tên file của guide là `README.md`; nội dung dùng cú pháp Markdown thông thường cho các công cụ repository preview.
