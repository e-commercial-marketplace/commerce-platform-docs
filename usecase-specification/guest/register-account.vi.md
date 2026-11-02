# Đăng ký tài khoản

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đăng ký tài khoản</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép người dùng tạo tài khoản Customer (mặc định) hoặc Seller. Tài khoản Seller mới ở trạng thái chờ duyệt.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tài khoản được tạo với mật khẩu đã băm và người dùng được đăng nhập ngay (nhận access token và refresh token).</li><li>Tài khoản Seller ở trạng thái PENDING cho tới khi Admin duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng đăng ký tài khoản.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị biểu mẫu đăng ký.</td></tr>
<tr><td>3. Nhập tên, email, mật khẩu và chọn loại tài khoản (Customer hoặc Seller).</td><td></td></tr>
<tr><td>4. Gửi yêu cầu đăng ký.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin hợp lệ, không đăng ký vai trò Admin và email chưa được dùng.</td></tr>
<tr><td></td><td>6. Tạo tài khoản với mật khẩu đã băm; Seller được đặt trạng thái chờ duyệt.</td></tr>
<tr><td></td><td>7. Cấp access token và refresh token rồi trả về kết quả.</td></tr>
<tr><td>8. Xem kết quả đăng ký.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Thông tin không hợp lệ, chọn vai trò Admin hoặc email đã được dùng tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin đăng ký chưa hợp lệ và không tạo tài khoản.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
