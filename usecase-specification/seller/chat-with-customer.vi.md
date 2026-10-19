# Trò chuyện với khách hàng

<table>
<tr><td colspan="2"><b>Tên use case:</b> Trò chuyện với khách hàng</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller trò chuyện với người tham gia cuộc trò chuyện.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập; dữ liệu thuộc về Seller.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Tin nhắn được lưu trong cuộc trò chuyện.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở cuộc trò chuyện.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị tin nhắn trước đó.</td></tr>
<tr><td>3. Nhập tin nhắn.</td><td></td></tr>
<tr><td>4. Gửi tin nhắn.</td><td></td></tr>
<tr><td></td><td>5. Lưu và gửi tin nhắn đến người tham gia còn lại.</td></tr>
<tr><td></td><td>6. Hiển thị tin nhắn mới.</td></tr>
<tr><td>7. Xem tin nhắn.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Không có.</b></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
