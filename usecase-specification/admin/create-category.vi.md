# Tạo danh mục

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tạo danh mục</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin tạo danh mục với tên chưa được dùng, không phân biệt hoa thường.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Danh mục được tạo và cache danh mục bị xóa.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở biểu mẫu tạo danh mục.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị biểu mẫu.</td></tr>
<tr><td>3. Nhập tên danh mục.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu tạo.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra tên không trống và chưa được dùng.</td></tr>
<tr><td></td><td>6. Lưu danh mục và trả về kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Tên trống hoặc đã được dùng tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo tên chưa hợp lệ và không tạo danh mục.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
