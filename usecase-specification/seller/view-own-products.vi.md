# Xem sản phẩm của tôi

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xem sản phẩm của tôi</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller xem sản phẩm của chính mình, kể cả sản phẩm đã ẩn, có tìm theo từ khóa và lọc theo trạng thái.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở sản phẩm của tôi, có thể nhập từ khóa hoặc lọc trạng thái.</td><td></td></tr>
<tr><td></td><td>2. Trả về trang sản phẩm của Seller, sắp theo tên.</td></tr>
<tr><td>3. Chọn một sản phẩm.</td><td></td></tr>
<tr><td></td><td>4. Kiểm tra sản phẩm thuộc về Seller và trả về chi tiết.</td></tr>
<tr><td>5. Xem chi tiết sản phẩm.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Sản phẩm không tồn tại hoặc của Seller khác tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không xem được sản phẩm.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
