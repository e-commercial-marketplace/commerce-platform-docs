# Cập nhật trạng thái giao hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Cập nhật trạng thái giao hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller cập nhật trạng thái giao hàng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập; dữ liệu thuộc về Seller.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Trạng thái giao hàng được cập nhật; tồn kho, đơn hàng và COD được cập nhật theo trạng thái giao hàng.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng cập nhật trạng thái giao hàng.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị trạng thái giao hàng hiện tại và thao tác được phép.</td></tr>
<tr><td>3. Chọn thông tin hoặc thao tác cần xử lý.</td><td></td></tr>
<tr><td>4. Gửi hoặc xác nhận yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin, điều kiện xử lý và quyền thực hiện.</td></tr>
<tr><td></td><td>6. Cập nhật trạng thái; tính lại trạng thái đơn hàng và COD khi mọi đợt giao hàng bắt buộc đã giao.</td></tr>
<tr><td></td><td>7. Cập nhật tồn kho theo kết quả: không hoàn hàng khi giao thất bại; chỉ hoàn hàng đã gửi khi kiện hàng quay về kho.</td></tr>
<tr><td></td><td>8. Nếu hủy đợt giao hàng hợp lệ, hủy các mặt hàng chưa gửi, hoàn tồn kho đúng một lần và tính lại tổng đơn cùng khoản COD còn chờ.</td></tr>
<tr><td></td><td>9. Hiển thị kết quả cập nhật.</td></tr>
<tr><td>10. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Trạng thái mới không phù hợp, thiếu thông tin giao hàng bắt buộc, hoặc hủy sau khi đã gửi/đã thanh toán tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không thể cập nhật trạng thái.</td></tr>
<tr><td>5.1.2. Sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Không giao được kiện hàng tại bước 5.</b></td></tr>
<tr><td></td><td>5.2.1. Ghi nhận giao hàng thất bại; không hoàn tồn kho.</td></tr>
<tr><td>5.2.2. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 3: Kiện hàng đã được đưa về kho của Seller tại bước 5.</b></td></tr>
<tr><td></td><td>5.3.1. Ghi nhận kiện hàng đã về kho và hoàn số lượng hàng đã gửi.</td></tr>
<tr><td>5.3.2. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 4: Hủy đợt giao hàng đang ở trạng thái PENDING hoặc PREPARING khi khoản thanh toán chưa hoàn tất tại bước 5.</b></td></tr>
<tr><td></td><td>5.4.1. Hủy các mặt hàng thuộc đợt giao hàng và giải phóng hàng chưa gửi đúng một lần; tính lại tổng tiền và khoản COD còn chờ. Nếu không còn mặt hàng nào, hủy đơn hàng.</td></tr>
<tr><td>5.4.2. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
