# Đăng ký tài khoản

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đăng ký tài khoản</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép người dùng đăng ký tài khoản.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tài khoản mới được tạo và lưu trong hệ thống.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng đăng ký tài khoản.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị biểu mẫu đăng ký.</td></tr>
<tr><td>3. Nhập tên hiển thị, email và mật khẩu.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu đăng ký.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra các thông tin bắt buộc và tính hợp lệ của thông tin đăng ký.</td></tr>
<tr><td></td><td>6. Kiểm tra email chưa được sử dụng cho tài khoản khác.</td></tr>
<tr><td></td><td>7. Tạo và lưu tài khoản mới với mật khẩu được bảo vệ.</td></tr>
<tr><td></td><td>8. Trả về kết quả đăng ký thành công.</td></tr>
<tr><td>9. Xem kết quả đăng ký.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Thông tin bắt buộc bị thiếu hoặc thông tin đăng ký không hợp lệ tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin đăng ký chưa hợp lệ và không tạo tài khoản.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 4.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Email đã được sử dụng tại bước 6.</b></td></tr>
<tr><td></td><td>6.1.1. Thông báo email đã được sử dụng và không tạo tài khoản mới.</td></tr>
<tr><td>6.1.2. Xem thông báo, nhập email khác và quay lại bước 4.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Lỗi hệ thống khi tạo và lưu tài khoản tại bước 7.</b></td></tr>
<tr><td></td><td>7.1.1. Thông báo không thể hoàn tất đăng ký do lỗi hệ thống và kết thúc lần đăng ký này.</td></tr>
<tr><td>7.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
