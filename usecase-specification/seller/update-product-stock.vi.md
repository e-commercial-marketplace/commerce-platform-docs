# Cập nhật tồn kho

<table>
<tr><td colspan="2"><b>Tên use case:</b> Cập nhật tồn kho</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller đặt lại số tồn kho của sản phẩm của chính mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tồn kho được cập nhật và cache chi tiết bị xóa.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở sản phẩm và nhập tồn kho mới.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra tồn kho không âm và sản phẩm thuộc về Seller.</td></tr>
<tr><td></td><td>4. Lưu tồn kho mới.</td></tr>
<tr><td>5. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Tồn kho âm tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo tồn kho không hợp lệ.</td></tr>
<tr><td>3.1.2. Xem thông báo, sửa thông tin và quay lại bước 1.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Sản phẩm không tồn tại hoặc của Seller khác tại bước 3.</b></td></tr>
<tr><td></td><td>3.2.1. Thông báo không cập nhật được.</td></tr>
<tr><td>3.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
