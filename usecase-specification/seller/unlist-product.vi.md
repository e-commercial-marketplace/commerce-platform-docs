# Ẩn sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Ẩn sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller ẩn sản phẩm của chính mình. Đây là xóa mềm: dòng dữ liệu vẫn còn để giữ lịch sử đơn hàng, và Seller có thể bán lại bằng chức năng sửa sản phẩm.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Sản phẩm có active = false, không còn hiện ở catalog công khai và cache chi tiết bị xóa.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn ẩn một sản phẩm của tôi.</td><td></td></tr>
<tr><td>2. Xác nhận ẩn.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra sản phẩm thuộc về Seller.</td></tr>
<tr><td></td><td>4. Đặt sản phẩm ở trạng thái ẩn.</td></tr>
<tr><td>5. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Sản phẩm không tồn tại hoặc của Seller khác tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không ẩn được sản phẩm.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
