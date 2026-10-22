# Đăng nhập bằng Google

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đăng nhập bằng Google</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Guest đăng nhập bằng tài khoản Google; nếu chưa có tài khoản thì hệ thống tạo tài khoản Customer mới.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li><li>Người dùng có tài khoản Google.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tài khoản được đăng nhập khi Google xác nhận danh tính hợp lệ.</li><li>Nếu chưa có tài khoản thì một tài khoản Customer mới được tạo và liên kết với tài khoản Google.</li><li>Nếu email đã có tài khoản thì tài khoản đó được liên kết với tài khoản Google.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng đăng nhập.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị tùy chọn đăng nhập bằng Google.</td></tr>
<tr><td>3. Chọn đăng nhập bằng Google.</td><td></td></tr>
<tr><td>4. Chọn tài khoản Google và đồng ý cấp quyền.</td><td></td></tr>
<tr><td>5. Gửi mã xác thực do Google cấp (ID token) đến hệ thống.</td><td></td></tr>
<tr><td></td><td>6. Xác minh mã xác thực với Google: chữ ký, nơi phát hành, ứng dụng nhận và thời hạn.</td></tr>
<tr><td></td><td>7. Kiểm tra email đã được Google xác minh.</td></tr>
<tr><td></td><td>8. Tìm tài khoản theo định danh Google hoặc theo email. Chưa có thì tạo tài khoản Customer mới; đã có theo email nhưng chưa liên kết thì liên kết với tài khoản Google.</td></tr>
<tr><td></td><td>9. Kiểm tra tài khoản đang hoạt động.</td></tr>
<tr><td></td><td>10. Đăng nhập tài khoản và trả về kết quả.</td></tr>
<tr><td>11. Xem kết quả đăng nhập.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Guest hủy hoặc không đồng ý cấp quyền tại bước 4.</b></td></tr>
<tr><td>4.1.1. Đóng cửa sổ đăng nhập Google.</td><td></td></tr>
<tr><td></td><td>4.1.2. Giữ nguyên màn hình đăng nhập và không tạo hay đăng nhập tài khoản nào.</td></tr>
<tr><td colspan="2"><b>Thay thế 2: Mã xác thực không hợp lệ hoặc đã hết hạn tại bước 6.</b></td></tr>
<tr><td></td><td>6.1.1. Thông báo đăng nhập bằng Google không thành công và không tạo hay liên kết tài khoản.</td></tr>
<tr><td>6.1.2. Xem thông báo và thử lại tại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 3: Email chưa được Google xác minh tại bước 7.</b></td></tr>
<tr><td></td><td>7.1.1. Thông báo đăng nhập bằng Google không thành công và không tạo hay liên kết tài khoản.</td></tr>
<tr><td>7.1.2. Xem thông báo và thử lại tại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 4: Tài khoản đã bị vô hiệu hóa tại bước 9.</b></td></tr>
<tr><td></td><td>9.1.1. Thông báo không thể đăng nhập bằng tài khoản này và không ghi nhận thay đổi nào.</td></tr>
<tr><td>9.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>5.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
