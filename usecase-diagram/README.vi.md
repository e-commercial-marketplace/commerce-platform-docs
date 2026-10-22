# Sơ đồ Use Case

Thư mục này chứa các sơ đồ use-case draw.io có thể chỉnh sửa cho các tác nhân của marketplace.

## Tệp

| Tệp | Tác nhân | Phạm vi |
|---|---|---|
| `guest-usecases.drawio` | UNAUTHENTICATED USER | Catalog công khai, chi tiết sản phẩm, đăng ký, đăng nhập, đăng nhập bằng Google và AI tìm kiếm sản phẩm cho guest |
| `customer-usecases.drawio` | CUSTOMER | Catalog/tìm kiếm/chi tiết công khai, giỏ hàng, checkout, thanh toán, đơn hàng, shipment, đánh giá, chat, đăng xuất và AI tìm kiếm của customer |
| `seller-usecases.drawio` | SELLER | Sản phẩm thuộc sở hữu, tồn kho, order item, shipment và chat với customer |
| `admin-usecases.drawio` | ADMIN | Người dùng, danh mục, voucher, kiểm duyệt, giám sát, cấu hình giao hàng và hoàn tiền |

## Quy tắc ký hiệu

- Đường liền nét: association, tác nhân tham gia vào use case. Chỉ các use case quản lý (manager) và độc lập nối với tác nhân; các lá CRUD nối qua `«extend»`.
- Mũi tên đứt nét nhãn `«include»`: use case nguồn yêu cầu khả năng của use case đích.
- Trong dự án này, `«include» Login` là cách viết tắt cho điều kiện tiên quyết xác thực; không có nghĩa người dùng đã xác thực phải đăng nhập lại cho mỗi request.
- `«include» Login` được vẽ từ mỗi use case quản lý / độc lập đã xác thực. Các lá CRUD được bao hàm bắc cầu qua `«extend»` tới manager.
- `Logout` chỉ được vẽ trong sơ đồ Customer. Seller và Admin dùng chung use case và endpoint này nên không có sơ đồ riêng.
- Mũi tên đứt nét nhãn `«extend»`: trỏ từ phần mở rộng (lá CRUD) tới use case cơ sở (manager), theo UML; cho thấy các thao tác CRUD thuộc về manager nào.
- Các use case catalog công khai và AI tìm kiếm công khai chủ ý không include `Login`.

## Hướng dẫn phong cách (khớp mẫu `usecase-admin_new`)

- Ranh giới hệ thống: hình chữ nhật `#f8fafc` phẳng, góc vuông, viền 1px `#334155`, tiêu đề căn giữa phía trên (`Guest/Customer/Seller/Admin Use-Case`, 16pt).
- Tác nhân: actor UML chuẩn bên dưới ranh giới, nét 1.5px, nhãn regular 14pt.
- Use case: ellipse cao 60px; manager / `Login` tô `#79CFF4` với nét đen 2px; lá và use case độc lập nền trắng với nét `#333333` 2px; chữ 14pt căn giữa.
- Association: nét liền 1.5px `#334155`, đi trực giao qua trục chung bên dưới các use case và các đường biên bên (không đường nào cắt ellipse use-case).
- `«include»` / `«extend»`: mũi tên đứt `5 5` màu xám `#475569` với đầu mũi tên đặc, nhãn 12pt, đi qua các kênh bus dọc giữa các cột.
- Mở trong draw.io qua File &gt; Open; kích thước trang được đặt cho từng sơ đồ, xuất ở 100%.

Các sơ đồ mô tả hành vi hệ thống dự kiến. Các tính năng UI hoãn lại chỉ hiển thị khi thuộc trách nhiệm role của backend; ranh giới MVP được ghi trong `PRD.vi.md` và SRS.
