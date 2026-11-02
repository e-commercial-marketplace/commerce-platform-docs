# Xem người dùng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xem người dùng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin xem danh sách người dùng, mới nhất trước, có lọc theo vai trò và tìm theo email hoặc tên, và xem chi tiết một người.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở danh sách người dùng, có thể chọn vai trò hoặc nhập từ khóa.</td><td></td></tr>
<tr><td></td><td>2. Trả về trang người dùng, mới nhất trước.</td></tr>
<tr><td>3. Chọn một người dùng.</td><td></td></tr>
<tr><td></td><td>4. Trả về chi tiết người dùng.</td></tr>
<tr><td>5. Xem chi tiết.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Người dùng không tồn tại tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không tìm thấy.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
