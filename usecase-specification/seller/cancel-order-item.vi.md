# Hủy dòng hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Hủy dòng hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller hủy một dòng hàng của mình khi chưa vào đợt giao và đơn chưa thanh toán. Tồn kho được trả lại và tổng tiền đơn được tính lại.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Dòng hàng CANCELLED, tồn kho được cộng lại và tổng tiền đơn được tính lại.</li><li>Nếu mọi dòng đã hủy, đơn hàng CANCELLED; khoản thanh toán chờ được cập nhật.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn hủy một dòng hàng và nhập lý do (nếu có).</td><td></td></tr>
<tr><td>2. Gửi yêu cầu hủy.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra dòng hàng thuộc về Seller, chưa vào đợt giao, chưa hủy và đơn chưa thanh toán.</td></tr>
<tr><td></td><td>4. Hủy dòng hàng và cộng lại tồn kho.</td></tr>
<tr><td></td><td>5. Tính lại tổng tiền đơn và cập nhật khoản thanh toán còn chờ.</td></tr>
<tr><td>6. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Dòng hàng đã vào đợt giao hoặc đơn đã thanh toán tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hủy.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Dòng hàng không tồn tại, của Seller khác hoặc đã hủy tại bước 3.</b></td></tr>
<tr><td></td><td>3.2.1. Thông báo không hủy được.</td></tr>
<tr><td>3.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
