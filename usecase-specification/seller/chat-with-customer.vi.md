# Trò chuyện với Customer

<table>
<tr><td colspan="2"><b>Tên use case:</b> Trò chuyện với Customer</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller trò chuyện trực tiếp với Customer. Mỗi cặp Customer và Seller có đúng một cuộc trò chuyện; tin nhắn được lưu qua REST và đẩy ngay qua WebSocket.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tin nhắn được lưu đúng một lần (khóa chống gửi trùng) và được đẩy tới cả hai bên.</li><li>Tin nhắn của bên kia được đánh dấu đã đọc khi mở cuộc trò chuyện.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn một cuộc trò chuyện trong danh sách, hoặc mở mới với một Customer.</td><td></td></tr>
<tr><td></td><td>2. Tìm cuộc trò chuyện của cặp này, chưa có thì tạo mới.</td></tr>
<tr><td></td><td>3. Hiển thị tin nhắn, mới nhất trước, và đánh dấu tin nhắn của bên kia là đã đọc.</td></tr>
<tr><td>4. Nhập nội dung hoặc chọn ảnh.</td><td></td></tr>
<tr><td>5. Gửi tin nhắn.</td><td></td></tr>
<tr><td></td><td>6. Kiểm tra có nội dung hoặc ảnh thuộc Cloudinary của hệ thống, có khóa chống gửi trùng, và người gửi thuộc cuộc trò chuyện.</td></tr>
<tr><td></td><td>7. Lưu tin nhắn; khóa đã dùng thì trả lại tin nhắn cũ.</td></tr>
<tr><td></td><td>8. Đẩy tin nhắn tới cả hai bên qua WebSocket sau khi lưu.</td></tr>
<tr><td>9. Xem tin nhắn mới.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Người nhận không tồn tại hoặc không hợp lệ tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không tìm thấy người nhận.</td></tr>
<tr><td>2.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Tin nhắn trống, ảnh không hợp lệ, thiếu khóa chống gửi trùng, hoặc người gửi không thuộc cuộc trò chuyện tại bước 6.</b></td></tr>
<tr><td></td><td>6.1.1. Thông báo không gửi được tin nhắn.</td></tr>
<tr><td>6.1.2. Xem thông báo, sửa thông tin và quay lại bước 4.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>5.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
