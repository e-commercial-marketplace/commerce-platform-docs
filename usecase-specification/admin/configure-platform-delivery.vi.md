# Quản lý đơn vị vận chuyển

<table>
<tr><td colspan="2"><b>Tên use case:</b> Quản lý đơn vị vận chuyển</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin thêm, đổi tên, bật hoặc tắt các đơn vị vận chuyển mà Seller được chọn khi giao hàng qua nền tảng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Danh sách đơn vị vận chuyển được cập nhật. Seller chỉ thấy các đơn vị đang hoạt động.</li><li>Đơn vị bị tắt không bị xoá và vẫn hiển thị trên các đợt giao hàng đã dùng nó.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng quản lý đơn vị vận chuyển.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị tất cả đơn vị vận chuyển, kể cả đơn vị đã tắt.</td></tr>
<tr><td>3. Thêm đơn vị mới, hoặc chọn một đơn vị để đổi tên, bật hoặc tắt.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra tên không để trống và chưa được dùng cho đơn vị khác (không phân biệt hoa thường).</td></tr>
<tr><td></td><td>6. Lưu thay đổi của đơn vị vận chuyển.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Tên bị thiếu, quá dài hoặc trùng với đơn vị khác tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin chưa hợp lệ và không lưu thay đổi.</td></tr>
<tr><td>5.1.2. Sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
