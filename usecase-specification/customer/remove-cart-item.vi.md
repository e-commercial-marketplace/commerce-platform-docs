# Xóa sản phẩm khỏi giỏ

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xóa sản phẩm khỏi giỏ</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer xóa một dòng hàng khỏi giỏ của mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Dòng hàng bị xóa và giỏ hàng mới được trả về.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn xóa một dòng hàng.</td><td></td></tr>
<tr><td>2. Xác nhận xóa.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra dòng hàng thuộc về Customer.</td></tr>
<tr><td></td><td>4. Xóa dòng hàng.</td></tr>
<tr><td></td><td>5. Trả về giỏ hàng mới.</td></tr>
<tr><td>6. Xem giỏ hàng mới.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Dòng hàng không tồn tại hoặc của người khác tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể xóa.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
