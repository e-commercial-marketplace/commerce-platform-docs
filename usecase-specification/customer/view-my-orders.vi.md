# Xem đơn hàng của tôi

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xem đơn hàng của tôi</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer xem danh sách đơn hàng của mình, mới nhất trước, và chi tiết một đơn gồm dòng hàng, đợt giao và thanh toán.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở đơn hàng của tôi.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị trang đơn hàng của Customer, mới nhất trước.</td></tr>
<tr><td>3. Chọn một đơn hàng.</td><td></td></tr>
<tr><td></td><td>4. Kiểm tra đơn hàng thuộc về Customer và trả về chi tiết.</td></tr>
<tr><td>5. Xem chi tiết đơn hàng.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Đơn hàng không tồn tại hoặc của người khác tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không tìm thấy đơn hàng.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
