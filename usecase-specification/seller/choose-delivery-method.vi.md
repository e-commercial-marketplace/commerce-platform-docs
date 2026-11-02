# Chọn cách giao hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Chọn cách giao hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller chọn tự giao hoặc giao qua nền tảng khi tạo đợt giao. Giao qua nền tảng phải chọn một đơn vị vận chuyển đang hoạt động; tự giao thì không có đơn vị vận chuyển.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Cách giao và đơn vị vận chuyển được lưu cùng đợt giao.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở phần chọn cách giao khi tạo đợt giao.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị hai cách giao và các đơn vị vận chuyển đang hoạt động.</td></tr>
<tr><td>3. Chọn tự giao, hoặc giao qua nền tảng kèm đơn vị vận chuyển.</td><td></td></tr>
<tr><td></td><td>4. Kiểm tra lựa chọn khi tạo đợt giao.</td></tr>
<tr><td>5. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Tự giao mà có đơn vị vận chuyển, giao qua nền tảng mà thiếu đơn vị, hoặc đơn vị không hoạt động tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo lựa chọn chưa hợp lệ.</td></tr>
<tr><td>4.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
