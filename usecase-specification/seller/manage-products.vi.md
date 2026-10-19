# Quản lý sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Quản lý sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller quản lý các thông tin được quản lý.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập; dữ liệu thuộc về Seller.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Sản phẩm chỉ thay đổi khi thao tác được chọn hoàn tất thành công.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng quản lý sản phẩm.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị các thông tin được quản lý và các thao tác được phép.</td></tr>
<tr><td>3. Chọn thao tác cần thực hiện.</td><td></td></tr>
<tr><td></td><td>4. Mở chức năng tương ứng.</td></tr>
<tr><td>5. Thực hiện thao tác đã chọn.</td><td></td></tr>
<tr><td></td><td>6. Hiển thị kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Chưa có dữ liệu để hiển thị tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo chưa có dữ liệu.</td></tr>
<tr><td>2.1.2. Xem thông báo và kết thúc lần xử lý này.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
