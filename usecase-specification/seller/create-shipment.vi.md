# Tạo đợt giao

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tạo đợt giao</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller tạo một đợt giao từ các dòng hàng đã xác nhận của mình trong một đơn. Mỗi Seller có tối đa một đợt giao cho mỗi đơn.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Đợt giao PENDING được tạo, phí giao 0, các dòng hàng thuộc về đợt giao này.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn đơn hàng và các dòng hàng đã xác nhận.</td><td></td></tr>
<tr><td>2. Chọn cách giao và gửi yêu cầu tạo đợt giao.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra các dòng hàng của Seller, cùng đơn, đã xác nhận và chưa vào đợt giao.</td></tr>
<tr><td></td><td>4. Kiểm tra Seller chưa có đợt giao của đơn này và cách giao hợp lệ.</td></tr>
<tr><td></td><td>5. Lưu đợt giao.</td></tr>
<tr><td>6. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Dòng hàng không hợp lệ tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không tạo được đợt giao.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Đã có đợt giao của đơn này hoặc cách giao không hợp lệ tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không tạo được đợt giao.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
