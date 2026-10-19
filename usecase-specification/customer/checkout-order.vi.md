# Đặt hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đặt hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer đặt hàng từ giỏ hàng của mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập; dữ liệu thuộc về Customer.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Đơn hàng được tạo, tồn kho được cập nhật và các sản phẩm đã mua được xóa khỏi giỏ hàng.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở giỏ hàng và chọn đặt hàng.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị sản phẩm và phần nhập địa chỉ giao hàng.</td></tr>
<tr><td>3. Nhập địa chỉ giao hàng và chọn COD.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu đặt hàng.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra sản phẩm, số lượng còn lại và địa chỉ giao hàng.</td></tr>
<tr><td></td><td>6. Tạo đơn hàng và lưu thông tin sản phẩm tại thời điểm đặt.</td></tr>
<tr><td></td><td>7. Cập nhật tồn kho và xóa sản phẩm đã mua khỏi giỏ hàng.</td></tr>
<tr><td></td><td>8. Hiển thị đơn hàng đã tạo.</td></tr>
<tr><td>9. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Giỏ hàng trống hoặc sản phẩm không còn đủ số lượng tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không thể đặt hàng và không tạo đơn hàng.</td></tr>
<tr><td>5.1.2. Cập nhật giỏ hàng rồi quay lại bước 1.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
