# Xem chi tiết sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xem chi tiết sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer xem chi tiết một sản phẩm đang bán.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn một sản phẩm.</td><td></td></tr>
<tr><td></td><td>2. Tìm sản phẩm đang bán theo mã.</td></tr>
<tr><td></td><td>3. Trả về chi tiết sản phẩm.</td></tr>
<tr><td>4. Xem chi tiết sản phẩm.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Sản phẩm không tồn tại hoặc đã ẩn tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không tìm thấy sản phẩm.</td></tr>
<tr><td>2.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 1.</b></td></tr>
<tr><td></td><td>1.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>1.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
