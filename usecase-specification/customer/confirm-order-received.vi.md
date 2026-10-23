# Xác nhận đã nhận hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xác nhận đã nhận hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer xác nhận đã nhận đủ hàng của một đơn hàng đã giao, để hoàn tất đơn hàng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li><li>Đơn hàng thuộc về Customer và mọi đợt giao hàng cần thiết đã giao (đơn hàng ở trạng thái đã giao).</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Đơn hàng chuyển sang hoàn tất.</li><li>Nếu Customer không xác nhận, hệ thống tự hoàn tất đơn hàng sau 3 ngày kể từ lần giao cuối.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chi tiết đơn hàng đã giao.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị đơn hàng và thao tác xác nhận đã nhận hàng.</td></tr>
<tr><td>3. Chọn xác nhận đã nhận hàng.</td><td></td></tr>
<tr><td></td><td>4. Kiểm tra đơn hàng thuộc về Customer và đang ở trạng thái đã giao.</td></tr>
<tr><td></td><td>5. Chuyển đơn hàng sang hoàn tất.</td></tr>
<tr><td></td><td>6. Hiển thị kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Đơn hàng chưa giao xong, đã hoàn tất hoặc đã hủy tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo đơn hàng không thể xác nhận đã nhận và giữ nguyên trạng thái.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
