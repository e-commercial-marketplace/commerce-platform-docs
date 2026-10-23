# Thanh toán đơn hàng qua VNPAY

<table>
<tr><td colspan="2"><b>Tên use case:</b> Thanh toán đơn hàng qua VNPAY</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer thanh toán trước đơn hàng của mình qua cổng VNPAY, lần đầu ngay khi đặt hàng hoặc thanh toán lại khi lần trước thất bại.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li><li>Customer có đơn hàng chọn VNPAY, đơn đang chờ thanh toán và chưa quá 24 giờ kể từ lúc đặt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Thanh toán thành công: lần thanh toán được hoàn tất và đơn hàng chuyển sang đã thanh toán.</li><li>Thanh toán thất bại: lần thanh toán được ghi nhận thất bại, đơn hàng vẫn chờ thanh toán và có thể thanh toán lại.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn thanh toán đơn hàng qua VNPAY (khi đặt hàng hoặc từ chi tiết đơn hàng).</td><td></td></tr>
<tr><td></td><td>2. Kiểm tra đơn hàng thuộc về Customer, đang chờ thanh toán và còn trong hạn 24 giờ.</td></tr>
<tr><td></td><td>3. Đánh thất bại lần thanh toán VNPAY còn chờ trước đó (nếu có), tạo lần thanh toán mới với tổng tiền hiện tại và tạo link thanh toán có chữ ký.</td></tr>
<tr><td></td><td>4. Chuyển Customer sang trang thanh toán của VNPAY.</td></tr>
<tr><td>5. Nhập thông tin thẻ và xác nhận thanh toán trên trang VNPAY.</td><td></td></tr>
<tr><td></td><td>6. Nhận kết quả từ VNPAY và kiểm tra chữ ký, mã giao dịch và số tiền.</td></tr>
<tr><td></td><td>7. Ghi nhận thanh toán thành công và chuyển đơn hàng sang đã thanh toán.</td></tr>
<tr><td></td><td>8. Đưa Customer quay về ứng dụng và hiển thị kết quả thanh toán.</td></tr>
<tr><td>9. Xem kết quả thanh toán.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Đơn hàng không còn chờ thanh toán, không phải đơn VNPAY hoặc đã quá hạn 24 giờ tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo đơn hàng không thể thanh toán và không tạo lần thanh toán mới.</td></tr>
<tr><td>2.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Customer hủy hoặc thanh toán bị từ chối trên trang VNPAY tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Ghi nhận lần thanh toán thất bại; đơn hàng vẫn chờ thanh toán.</td></tr>
<tr><td>5.1.2. Xem kết quả thất bại và có thể thanh toán lại tại bước 1.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 3: Kết quả không hợp lệ (sai chữ ký, mã giao dịch không tồn tại hoặc sai số tiền) tại bước 6.</b></td></tr>
<tr><td></td><td>6.1.1. Không thay đổi trạng thái thanh toán và đơn hàng; thông báo không xác nhận được kết quả thanh toán.</td></tr>
<tr><td>6.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 4: Kết quả của lần thanh toán này đã được ghi nhận trước đó tại bước 6.</b></td></tr>
<tr><td></td><td>6.2.1. Giữ nguyên trạng thái đã ghi nhận và hiển thị trạng thái hiện tại tại bước 8.</td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi tạo lần thanh toán tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
