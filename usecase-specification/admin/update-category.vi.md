# Đổi tên danh mục

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đổi tên danh mục</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin đổi tên danh mục. Tên mới không được trùng với danh mục khác.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tên được đổi; cache danh mục và cache chi tiết sản phẩm bị xóa.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn một danh mục để đổi tên.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị tên hiện tại.</td></tr>
<tr><td>3. Nhập tên mới.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra danh mục tồn tại và tên mới không trùng danh mục khác.</td></tr>
<tr><td></td><td>6. Lưu tên mới và trả về kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Tên trống hoặc trùng danh mục khác tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo tên chưa hợp lệ.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Danh mục không tồn tại tại bước 5.</b></td></tr>
<tr><td></td><td>5.2.1. Thông báo không tìm thấy.</td></tr>
<tr><td>5.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
