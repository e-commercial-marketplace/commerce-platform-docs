# Kiểm duyệt đánh giá và vi phạm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Kiểm duyệt đánh giá và vi phạm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin kiểm duyệt đánh giá và trường hợp vi phạm.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Kết quả kiểm duyệt được lưu trong hệ thống.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng kiểm duyệt đánh giá và vi phạm.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị kết quả kiểm duyệt hiện tại và thao tác được phép.</td></tr>
<tr><td>3. Chọn thông tin hoặc thao tác cần xử lý.</td><td></td></tr>
<tr><td>4. Gửi hoặc xác nhận yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin, điều kiện xử lý và quyền thực hiện.</td></tr>
<tr><td></td><td>6. Cập nhật kết quả kiểm duyệt.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Không có.</b></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
