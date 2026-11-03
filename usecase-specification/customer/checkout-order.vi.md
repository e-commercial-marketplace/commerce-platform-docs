# Đặt hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đặt hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer đặt các dòng hàng đã chọn trong giỏ. Một đơn có thể gồm hàng của nhiều Seller. Toàn bộ chạy trong một giao dịch.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Đơn hàng PENDING được tạo với giá chụp tại lúc đặt, tồn kho bị trừ, lượt voucher bị trừ, khoản thanh toán được tạo và các dòng đã mua bị xóa khỏi giỏ.</li><li>Với VNPAY, phản hồi có đường dẫn thanh toán và đơn phải được trả trong 24 giờ.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn các dòng hàng và chọn đặt hàng.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị sản phẩm và phần nhập thông tin.</td></tr>
<tr><td>3. Chọn địa chỉ trong hồ sơ (nếu có) hoặc nhập địa chỉ giao hàng khác, chọn COD hoặc VNPAY và nhập mã voucher (nếu có).</td><td></td></tr>
<tr><td>4. Gửi yêu cầu đặt hàng.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra dòng hàng thuộc về Customer, sản phẩm đang bán và trừ tồn kho từng sản phẩm.</td></tr>
<tr><td></td><td>6. Kiểm tra và trừ một lượt voucher (nếu có).</td></tr>
<tr><td></td><td>7. Tạo đơn hàng với giá chụp lại và tính tổng tiền sau giảm (phí giao 0).</td></tr>
<tr><td></td><td>8. Tạo khoản thanh toán COD, hoặc khoản VNPAY kèm đường dẫn thanh toán.</td></tr>
<tr><td></td><td>9. Xóa các dòng đã mua khỏi giỏ và trả về đơn hàng.</td></tr>
<tr><td>10. Xem đơn hàng đã tạo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Dòng hàng không hợp lệ, sản phẩm đã ẩn hoặc không đủ hàng tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Hoàn tác toàn bộ, thông báo không thể đặt hàng và không tạo đơn.</td></tr>
<tr><td>5.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Mã voucher không tồn tại, ngoài thời hạn hoặc đã hết lượt tại bước 6.</b></td></tr>
<tr><td></td><td>6.1.1. Hoàn tác toàn bộ, thông báo voucher không dùng được và không tạo đơn.</td></tr>
<tr><td>6.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
