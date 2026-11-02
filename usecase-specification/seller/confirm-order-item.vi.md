# Xác nhận dòng hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xác nhận dòng hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller xác nhận một dòng hàng của mình. Đơn VNPAY phải được thanh toán trước khi xác nhận.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Dòng hàng chuyển sang CONFIRMED và có thể đưa vào đợt giao.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn xác nhận một dòng hàng.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu xác nhận.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra dòng hàng thuộc về Seller và đang chờ xử lý.</td></tr>
<tr><td></td><td>4. Với đơn VNPAY, kiểm tra đơn đã thanh toán.</td></tr>
<tr><td></td><td>5. Chuyển dòng hàng sang đã xác nhận.</td></tr>
<tr><td>6. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Dòng hàng không tồn tại, của Seller khác hoặc không còn chờ xử lý tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không xác nhận được.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Đơn VNPAY chưa thanh toán tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo đơn hàng chưa thanh toán.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
