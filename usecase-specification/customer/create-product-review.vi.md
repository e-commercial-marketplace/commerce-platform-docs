# Tạo đánh giá sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tạo đánh giá sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer tạo đánh giá sản phẩm khi đủ điều kiện.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập; dữ liệu thuộc về Customer.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Đánh giá sản phẩm được lưu trong hệ thống.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng đánh giá sản phẩm.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị sản phẩm đủ điều kiện đánh giá.</td></tr>
<tr><td>3. Chọn sản phẩm đã mua và nhận, nhập điểm và bình luận.</td><td></td></tr>
<tr><td>4. Gửi đánh giá.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra điều kiện đánh giá và điểm từ 1 đến 5.</td></tr>
<tr><td></td><td>6. Lưu đánh giá.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Chưa nhận sản phẩm, đã đánh giá hoặc điểm ngoài khoảng 1 đến 5 tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo chưa thể tạo đánh giá.</td></tr>
<tr><td>5.1.2. Sửa thông tin hoặc chọn sản phẩm đủ điều kiện rồi quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
