# Sửa người dùng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Sửa người dùng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin sửa tên, liên hệ, vai trò, trạng thái Seller và trạng thái hoạt động. Admin không thể tự đổi vai trò hay tự vô hiệu hóa chính mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Người dùng được cập nhật. Khi bị vô hiệu hóa, mọi refresh token của họ bị thu hồi.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở một người dùng để sửa.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị thông tin hiện tại.</td></tr>
<tr><td>3. Sửa thông tin cần đổi.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra người dùng tồn tại, thông tin hợp lệ, không tự khóa chính mình và trạng thái Seller chỉ dùng cho Seller.</td></tr>
<tr><td></td><td>6. Lưu thay đổi và trả về kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Thông tin không hợp lệ, tự khóa chính mình, hoặc đặt trạng thái Seller cho người không phải Seller tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không lưu được.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Người dùng không tồn tại tại bước 5.</b></td></tr>
<tr><td></td><td>5.2.1. Thông báo không tìm thấy.</td></tr>
<tr><td>5.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
