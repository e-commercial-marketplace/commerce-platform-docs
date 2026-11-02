# Thanh toán khi nhận hàng (COD)

<table>
<tr><td colspan="2"><b>Tên use case:</b> Thanh toán khi nhận hàng (COD)</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Customer chọn COD khi đặt hàng và trả tiền khi nhận hàng. Không có API riêng để trả COD: khoản thanh toán tự hoàn tất khi mọi dòng hàng chưa hủy của đơn đã được giao.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đặt đơn hàng COD.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Khi mọi dòng hàng chưa hủy đã giao: đơn hàng DELIVERED và khoản COD COMPLETED.</li><li>Khoản COD đổi số tiền khi có dòng hàng bị hủy, và thất bại nếu cả đơn bị hủy.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn COD khi đặt hàng.</td><td></td></tr>
<tr><td></td><td>2. Tạo khoản thanh toán COD ở trạng thái chờ thu.</td></tr>
<tr><td>3. Nhận kiện hàng và trả tiền cho người giao.</td><td></td></tr>
<tr><td></td><td>4. Ghi nhận đợt giao đã giao khi Seller cập nhật trạng thái.</td></tr>
<tr><td></td><td>5. Kiểm tra mọi dòng hàng chưa hủy của đơn đã nằm trong đợt giao đã giao.</td></tr>
<tr><td></td><td>6. Chuyển đơn hàng sang đã giao và hoàn tất khoản COD.</td></tr>
<tr><td>7. Xem trạng thái đơn hàng và thanh toán.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Còn dòng hàng chưa được giao tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Giữ khoản thanh toán ở trạng thái chờ thu và đơn hàng chưa đổi.</td></tr>
<tr><td>5.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
