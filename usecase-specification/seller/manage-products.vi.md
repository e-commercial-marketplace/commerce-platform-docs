# Quản lý sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Quản lý sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller quản lý sản phẩm của chính mình: xem, tạo, sửa, ẩn và cập nhật tồn kho.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Sản phẩm chỉ đổi khi thao tác thành công.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở sản phẩm của tôi.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị danh sách sản phẩm của Seller, kể cả sản phẩm đã ẩn.</td></tr>
<tr><td>3. Chọn tạo, sửa, ẩn hoặc cập nhật tồn kho.</td><td></td></tr>
<tr><td></td><td>4. Thực hiện thao tác đã chọn.</td></tr>
<tr><td>5. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2">Không có.</td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
