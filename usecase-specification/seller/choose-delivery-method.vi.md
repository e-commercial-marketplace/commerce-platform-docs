# Chọn phương thức giao hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Chọn phương thức giao hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller chọn phương thức giao hàng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập; dữ liệu thuộc về Seller.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Phương thức giao hàng được lưu cho đợt giao hàng.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng chọn phương thức giao hàng.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị phương thức giao hàng hiện tại và thao tác được phép.</td></tr>
<tr><td>3. Chọn giao qua nền tảng (kèm một đơn vị vận chuyển đang hoạt động do Admin quản lý) hoặc tự giao (ví dụ đơn gần, không chọn đơn vị).</td><td></td></tr>
<tr><td>4. Gửi hoặc xác nhận yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin, điều kiện xử lý và quyền thực hiện.</td></tr>
<tr><td></td><td>6. Cập nhật phương thức giao hàng.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Giao qua nền tảng mà không chọn đơn vị hoặc đơn vị đã tắt, hoặc tự giao mà vẫn chọn đơn vị, tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo đơn vị vận chuyển không hợp lệ và không lưu.</td></tr>
<tr><td>5.1.2. Chọn lại và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
