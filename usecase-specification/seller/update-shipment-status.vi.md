# Cập nhật trạng thái đợt giao

<table>
<tr><td colspan="2"><b>Tên use case:</b> Cập nhật trạng thái đợt giao</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller chuyển trạng thái đợt giao của mình: PENDING sang PREPARING hoặc CANCELLED, PREPARING sang SHIPPED hoặc CANCELLED, SHIPPED sang DELIVERED hoặc FAILED, FAILED sang RETURNED.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>SHIPPED lưu mã vận đơn. DELIVERED: nếu mọi dòng chưa hủy đã giao thì đơn DELIVERED và COD hoàn tất.</li><li>RETURNED trả lại tồn kho, FAILED thì không. CANCELLED hủy các dòng hàng, trả kho và tính lại tổng tiền.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn một đợt giao và trạng thái mới, nhập mã vận đơn khi chuyển sang SHIPPED.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra đợt giao thuộc về Seller và trạng thái mới được phép.</td></tr>
<tr><td></td><td>4. Lưu trạng thái mới.</td></tr>
<tr><td></td><td>5. Áp dụng hệ quả lên đơn hàng, thanh toán và tồn kho theo trạng thái mới.</td></tr>
<tr><td>6. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Chuyển trạng thái không được phép, thiếu mã vận đơn, hoặc hủy đợt giao của đơn đã thanh toán tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không cập nhật được và giữ nguyên trạng thái.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Đợt giao không tồn tại hoặc của Seller khác tại bước 3.</b></td></tr>
<tr><td></td><td>3.2.1. Thông báo không cập nhật được.</td></tr>
<tr><td>3.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
