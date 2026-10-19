# Đăng nhập

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đăng nhập</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Guest đăng nhập vào tài khoản của mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tài khoản được đăng nhập khi thông tin chính xác.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng đăng nhập.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị biểu mẫu đăng nhập.</td></tr>
<tr><td>3. Nhập email và mật khẩu.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu đăng nhập.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin đăng nhập.</td></tr>
<tr><td></td><td>6. Đăng nhập tài khoản và trả về kết quả.</td></tr>
<tr><td>7. Xem kết quả đăng nhập.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Email hoặc mật khẩu không chính xác tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin đăng nhập không chính xác.</td></tr>
<tr><td>5.1.2. Xem thông báo và nhập lại thông tin tại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
