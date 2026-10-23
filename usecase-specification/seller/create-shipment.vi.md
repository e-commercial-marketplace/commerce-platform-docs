# Tạo đợt giao hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tạo đợt giao hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller tạo đợt giao hàng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập; dữ liệu thuộc về Seller.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Đợt giao hàng mới được tạo cho các mặt hàng đủ điều kiện.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng tạo đợt giao hàng.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị đợt giao hàng hiện tại và thao tác được phép.</td></tr>
<tr><td>3. Chọn các mặt hàng, cách giao (qua nền tảng hoặc tự giao) và đơn vị vận chuyển nếu giao qua nền tảng.</td><td></td></tr>
<tr><td>4. Gửi hoặc xác nhận yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin, điều kiện xử lý và quyền thực hiện.</td></tr>
<tr><td></td><td>6. Tạo và lưu đợt giao hàng mới ở trạng thái chờ xử lý.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Có mặt hàng chưa được xác nhận, đã thuộc đợt giao hàng khác, hoặc khác đơn hàng/người bán tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không thể tạo đợt giao hàng.</td></tr>
<tr><td>5.1.2. Chọn lại các mặt hàng và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Giao qua nền tảng mà không chọn đơn vị vận chuyển hoặc đơn vị đã tắt, hoặc tự giao mà vẫn chọn đơn vị, tại bước 5.</b></td></tr>
<tr><td></td><td>5.2.1. Thông báo đơn vị vận chuyển không hợp lệ và không tạo đợt giao hàng.</td></tr>
<tr><td>5.2.2. Chọn lại cách giao hoặc đơn vị và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 3: Seller đã có đợt giao hàng cho đơn hàng này tại bước 5.</b></td></tr>
<tr><td></td><td>5.3.1. Thông báo mỗi đơn hàng chỉ có một đợt giao hàng của Seller và không tạo thêm.</td></tr>
<tr><td>5.3.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
