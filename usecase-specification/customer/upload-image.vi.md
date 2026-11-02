# Tải ảnh lên

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tải ảnh lên</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cấp chữ ký để Customer tải ảnh trong tin nhắn thẳng lên Cloudinary, không đi qua backend. Use case này được dùng bên trong chức năng cần ảnh.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Cloudinary.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Ảnh nằm trên Cloudinary và ứng dụng có đường dẫn ảnh để dùng làm ảnh trong tin nhắn.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn ảnh từ thiết bị.</td><td></td></tr>
<tr><td></td><td>2. Kiểm tra người gọi được phép tải ảnh cho mục đích này và hệ thống đã cấu hình Cloudinary.</td></tr>
<tr><td></td><td>3. Sinh mã ảnh ngẫu nhiên, ký các tham số bằng khóa bí mật và trả chữ ký có hiệu lực một giờ.</td></tr>
<tr><td></td><td>4. Ứng dụng gửi tệp ảnh kèm chữ ký lên Cloudinary.</td></tr>
<tr><td></td><td>5. Nhận đường dẫn ảnh từ Cloudinary.</td></tr>
<tr><td>6. Xem ảnh đã chọn.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Người gọi không được phép hoặc hệ thống chưa cấu hình Cloudinary tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể tải ảnh lên.</td></tr>
<tr><td>2.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Cloudinary từ chối tệp (sai định dạng hoặc chữ ký hết hạn) tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo tải ảnh thất bại và cho chọn lại ảnh.</td></tr>
<tr><td>4.1.2. Xem thông báo và quay lại bước 1.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 1.</b></td></tr>
<tr><td></td><td>1.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>1.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
