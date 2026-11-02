# Xem giỏ hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xem giỏ hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer xem các dòng hàng của mình, mới thêm trước. Giá và tồn kho luôn là giá trị hiện tại của sản phẩm.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở giỏ hàng.</td><td></td></tr>
<tr><td></td><td>2. Đọc các dòng hàng của Customer và giá, tồn kho hiện tại.</td></tr>
<tr><td>3. Xem giỏ hàng.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2">Không có.</td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 1.</b></td></tr>
<tr><td></td><td>1.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>1.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
