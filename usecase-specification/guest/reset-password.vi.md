# Đặt lại mật khẩu

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đặt lại mật khẩu</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép người dùng quên mật khẩu nhận mã 6 số qua email rồi đặt mật khẩu mới. Áp dụng cho mọi vai trò.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Dịch vụ email (Brevo).</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Mật khẩu được thay, mã đã dùng hết hiệu lực và mọi refresh token bị thu hồi; người dùng đăng nhập lại bằng mật khẩu mới.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn quên mật khẩu và nhập email.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu lấy mã.</td><td></td></tr>
<tr><td></td><td>3. Nếu email thuộc tài khoản đang hoạt động và chưa vượt giới hạn (60 giây giữa hai lần, 5 mã mỗi giờ), tạo mã 6 số hiệu lực 15 phút và gửi email.</td></tr>
<tr><td></td><td>4. Luôn trả cùng một phản hồi thành công, kể cả khi email không tồn tại.</td></tr>
<tr><td>5. Nhập email, mã và mật khẩu mới.</td><td></td></tr>
<tr><td>6. Gửi yêu cầu đặt lại mật khẩu.</td><td></td></tr>
<tr><td></td><td>7. Kiểm tra mã mới nhất còn hạn, chưa dùng, sai chưa quá 5 lần và đúng.</td></tr>
<tr><td></td><td>8. Đổi mật khẩu, vô hiệu mã và thu hồi mọi refresh token.</td></tr>
<tr><td>9. Xem kết quả và đăng nhập lại.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Mã sai, hết hạn, đã dùng hoặc đã sai quá số lần cho phép tại bước 7.</b></td></tr>
<tr><td></td><td>7.1.1. Thông báo cùng một lỗi cho mọi trường hợp; mã sai được tính thêm một lần thử.</td></tr>
<tr><td>7.1.2. Xem thông báo, sửa thông tin và quay lại bước 5.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 6.</b></td></tr>
<tr><td></td><td>6.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>6.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
