# Duyệt hoặc tạm ngưng Seller

<table>
<tr><td colspan="2"><b>Tên use case:</b> Duyệt hoặc tạm ngưng Seller</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin duyệt Seller đang chờ hoặc đang tạm ngưng, và tạm ngưng Seller đã duyệt. Seller chưa duyệt không dùng được các chức năng bán hàng và không có trang shop.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Trạng thái Seller được cập nhật và có hiệu lực ngay ở request tiếp theo, vì được đọc từ cơ sở dữ liệu.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở danh sách Seller.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị Seller và trạng thái hiện tại.</td></tr>
<tr><td>3. Chọn duyệt hoặc tạm ngưng một Seller.</td><td></td></tr>
<tr><td>4. Xác nhận.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra người này là Seller và trạng thái hiện tại cho phép thao tác.</td></tr>
<tr><td></td><td>6. Lưu trạng thái mới và xóa cache trang shop.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Không phải Seller, đã được duyệt, hoặc tạm ngưng khi chưa được duyệt tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không đổi được trạng thái.</td></tr>
<tr><td>5.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Người dùng không tồn tại tại bước 5.</b></td></tr>
<tr><td></td><td>5.2.1. Thông báo không tìm thấy.</td></tr>
<tr><td>5.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
