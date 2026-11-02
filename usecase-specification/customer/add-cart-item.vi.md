# Thêm sản phẩm vào giỏ

<table>
<tr><td colspan="2"><b>Tên use case:</b> Thêm sản phẩm vào giỏ</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer thêm sản phẩm đang bán vào giỏ. Sản phẩm đã có trong giỏ thì được cộng dồn số lượng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Dòng hàng được tạo hoặc cộng dồn và giỏ hàng mới được trả về.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn sản phẩm và số lượng.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu thêm vào giỏ.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra sản phẩm tồn tại, đang bán và đủ hàng cho tổng số lượng trong giỏ.</td></tr>
<tr><td></td><td>4. Tạo dòng hàng mới hoặc cộng dồn vào dòng cũ.</td></tr>
<tr><td></td><td>5. Trả về giỏ hàng mới.</td></tr>
<tr><td>6. Xem giỏ hàng mới.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Sản phẩm không tồn tại, đã ẩn hoặc không đủ hàng tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể thêm và không đổi giỏ hàng.</td></tr>
<tr><td>3.1.2. Xem thông báo, sửa thông tin và quay lại bước 1.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
