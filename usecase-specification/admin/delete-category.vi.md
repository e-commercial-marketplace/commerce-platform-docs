# Xóa danh mục

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xóa danh mục</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin xóa hẳn một danh mục không còn sản phẩm nào, kể cả sản phẩm đã ẩn.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Danh mục bị xóa và cache danh mục bị xóa.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn xóa một danh mục.</td><td></td></tr>
<tr><td>2. Xác nhận.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra danh mục tồn tại và không còn sản phẩm.</td></tr>
<tr><td></td><td>4. Xóa danh mục.</td></tr>
<tr><td>5. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Danh mục không tồn tại hoặc còn sản phẩm tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không xóa được danh mục.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
