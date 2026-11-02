# Vô hiệu hóa người dùng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Vô hiệu hóa người dùng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin vô hiệu hóa một tài khoản. Đây là xóa mềm: dòng dữ liệu vẫn còn. Admin không thể tự vô hiệu hóa chính mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tài khoản không đăng nhập được và mọi refresh token bị thu hồi. Access token hiện tại hết hạn tự nhiên.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn vô hiệu hóa một người dùng.</td><td></td></tr>
<tr><td>2. Xác nhận.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra người dùng tồn tại và không phải chính Admin.</td></tr>
<tr><td></td><td>4. Vô hiệu hóa tài khoản và thu hồi mọi refresh token.</td></tr>
<tr><td>5. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Người dùng không tồn tại hoặc là chính Admin tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không vô hiệu hóa được.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
