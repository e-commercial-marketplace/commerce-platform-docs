# Đổi mật khẩu

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đổi mật khẩu</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép người dùng đã đăng nhập đổi mật khẩu của chính mình. Use case này dùng chung cho Customer, Seller và Admin.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Mật khẩu được thay, mọi refresh token cũ bị thu hồi và người dùng nhận cặp token mới.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng đổi mật khẩu.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị biểu mẫu đổi mật khẩu.</td></tr>
<tr><td>3. Nhập mật khẩu hiện tại và mật khẩu mới.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu đổi mật khẩu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra mật khẩu hiện tại đúng và mật khẩu mới khác mật khẩu cũ.</td></tr>
<tr><td></td><td>6. Lưu mật khẩu mới, thu hồi mọi refresh token và cấp cặp token mới.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Mật khẩu hiện tại sai hoặc mật khẩu mới trùng mật khẩu cũ tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo mật khẩu chưa hợp lệ và không đổi gì.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
