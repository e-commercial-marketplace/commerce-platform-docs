# Xóa sản phẩm khỏi giỏ hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xóa sản phẩm khỏi giỏ hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer thay đổi giỏ hàng của mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập; dữ liệu thuộc về Customer.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Sản phẩm được xóa khỏi giỏ hàng.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở giỏ hàng hoặc catalog sản phẩm.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị các sản phẩm liên quan.</td></tr>
<tr><td>3. Chọn sản phẩm cần xóa.</td><td></td></tr>
<tr><td>4. Xác nhận thay đổi giỏ hàng.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra sản phẩm và số lượng còn lại.</td></tr>
<tr><td></td><td>6. Cập nhật giỏ hàng và tính lại số tiền.</td></tr>
<tr><td></td><td>7. Hiển thị giỏ hàng mới.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Không có.</b></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
