# Đổi số lượng trong giỏ

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đổi số lượng trong giỏ</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer đổi số lượng của một dòng hàng trong giỏ của mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Số lượng được cập nhật và giỏ hàng mới được trả về.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Nhập số lượng mới cho một dòng hàng.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra dòng hàng thuộc về Customer, sản phẩm đang bán và đủ hàng.</td></tr>
<tr><td></td><td>4. Lưu số lượng mới.</td></tr>
<tr><td></td><td>5. Trả về giỏ hàng mới.</td></tr>
<tr><td>6. Xem giỏ hàng mới.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Dòng hàng không tồn tại hoặc của người khác, sản phẩm đã ẩn, hoặc không đủ hàng tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể cập nhật và không đổi giỏ hàng.</td></tr>
<tr><td>3.1.2. Xem thông báo, sửa thông tin và quay lại bước 1.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
