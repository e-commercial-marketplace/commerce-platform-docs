# Duyệt catalog công khai

<table>
<tr><td colspan="2"><b>Tên use case:</b> Duyệt catalog công khai</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Guest xem thông tin.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Dữ liệu sản phẩm không thay đổi khi được xem.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng duyệt catalog công khai.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị thông tin tác nhân được phép xem.</td></tr>
<tr><td>3. Chọn thông tin cần xem.</td><td></td></tr>
<tr><td></td><td>4. Hiển thị chi tiết thông tin.</td></tr>
<tr><td>5. Xem thông tin.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Không có dữ liệu để hiển thị tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo chưa có dữ liệu phù hợp.</td></tr>
<tr><td>2.1.2. Xem thông báo hoặc quay lại danh sách.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
