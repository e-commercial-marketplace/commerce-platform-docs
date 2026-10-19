# Duyệt hoặc tạm ngưng người bán

<table>
<tr><td colspan="2"><b>Tên use case:</b> Duyệt hoặc tạm ngưng người bán</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin xem xét trạng thái người bán.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Trạng thái người bán được cập nhật theo quyết định của Admin.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng duyệt hoặc tạm ngưng người bán.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị trạng thái người bán hiện tại và thao tác được phép.</td></tr>
<tr><td>3. Chọn thông tin hoặc thao tác cần xử lý.</td><td></td></tr>
<tr><td>4. Gửi hoặc xác nhận yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin, điều kiện xử lý và quyền thực hiện.</td></tr>
<tr><td></td><td>6. Cập nhật trạng thái người bán.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Trạng thái người bán đã thay đổi tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không thể thay đổi trạng thái.</td></tr>
<tr><td>5.1.2. Tải lại danh sách và xem xét lại.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
