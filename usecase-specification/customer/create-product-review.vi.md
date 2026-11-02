# Đánh giá sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Đánh giá sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer đánh giá sản phẩm đang bán đã mua trong một đơn hàng hoàn tất. Mỗi Customer chỉ đánh giá một sản phẩm một lần.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Đánh giá được lưu và điểm trung bình, số lượt đánh giá của sản phẩm được cập nhật.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng đánh giá của một sản phẩm đã mua.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị biểu mẫu đánh giá.</td></tr>
<tr><td>3. Chọn điểm từ 1 đến 5 và nhập bình luận.</td><td></td></tr>
<tr><td>4. Gửi đánh giá.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra sản phẩm đang bán, đã mua trong đơn hoàn tất và chưa đánh giá.</td></tr>
<tr><td></td><td>6. Lưu đánh giá và cập nhật điểm của sản phẩm.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Sản phẩm đã ẩn, chưa có đơn hoàn tất chứa sản phẩm, đã đánh giá, hoặc thông tin không hợp lệ tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo chưa thể tạo đánh giá.</td></tr>
<tr><td>5.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
