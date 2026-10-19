# Tạo mã giảm giá

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tạo mã giảm giá</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin tạo thông tin mới.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Mã giảm giá mới được lưu trong hệ thống.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng tạo mã giảm giá.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị thông tin mới hiện tại và thao tác được phép.</td></tr>
<tr><td>3. Nhập thông tin cần tạo.</td><td></td></tr>
<tr><td>4. Gửi hoặc xác nhận yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin, điều kiện xử lý và quyền thực hiện.</td></tr>
<tr><td></td><td>6. Tạo và lưu thông tin mới.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Thông tin bị thiếu hoặc không hợp lệ tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin chưa hợp lệ và không lưu thay đổi.</td></tr>
<tr><td>5.1.2. Sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
