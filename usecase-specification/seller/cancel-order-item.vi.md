# Hủy mặt hàng trong đơn hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Hủy mặt hàng trong đơn hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller hủy mặt hàng trong đơn hàng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập; dữ liệu thuộc về Seller.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Mặt hàng được hủy, số lượng chưa gửi được hoàn đúng một lần và tổng tiền chưa thanh toán được tính lại khi cần.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng hủy mặt hàng trong đơn hàng.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị mặt hàng trong đơn hàng hiện tại và thao tác được phép.</td></tr>
<tr><td>3. Chọn thông tin hoặc thao tác cần xử lý.</td><td></td></tr>
<tr><td>4. Gửi hoặc xác nhận yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin, điều kiện xử lý và quyền thực hiện.</td></tr>
<tr><td></td><td>6. Hủy mặt hàng, hoàn số lượng chưa gửi đúng một lần và tính lại tổng đơn hàng cùng khoản thanh toán còn chờ khi cần.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Mặt hàng đã được đưa vào đợt giao hàng hoặc không đủ điều kiện tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không thể hủy mặt hàng.</td></tr>
<tr><td>5.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
