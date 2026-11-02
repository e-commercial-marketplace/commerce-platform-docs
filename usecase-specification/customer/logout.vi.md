# Đăng xuất

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đăng xuất</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép người dùng đã đăng nhập kết thúc phiên bằng cách thu hồi refresh token của chính mình. Use case này dùng chung cho Customer, Seller và Admin.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng đã đăng nhập và có refresh token còn hiệu lực.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Refresh token bị thu hồi. Access token hiện tại hết hạn tự nhiên (tối đa 15 phút).</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn đăng xuất.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu đăng xuất kèm refresh token.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra refresh token còn hiệu lực và thuộc về người gọi.</td></tr>
<tr><td></td><td>4. Thu hồi refresh token.</td></tr>
<tr><td>5. Xem kết quả đăng xuất.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Refresh token không tồn tại, đã bị thu hồi hoặc không thuộc về người gọi tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo cùng một lỗi để không lộ thông tin và không thu hồi token nào.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
