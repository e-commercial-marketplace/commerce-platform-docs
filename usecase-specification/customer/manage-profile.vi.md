# Quản lý hồ sơ

<table>
<tr><td colspan="2"><b>Tên use case:</b> Quản lý hồ sơ</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép người dùng đã đăng nhập xem và sửa tên, số điện thoại, địa chỉ của chính mình. Use case này dùng chung cho Customer, Seller và Admin.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Hồ sơ được cập nhật. Email, vai trò và trạng thái không đổi được ở đây.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở hồ sơ của tôi.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị tên, email, số điện thoại và địa chỉ.</td></tr>
<tr><td>3. Sửa tên, số điện thoại hoặc địa chỉ.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra thông tin: tên không trống, số điện thoại Việt Nam hợp lệ, địa chỉ tối đa 500 ký tự.</td></tr>
<tr><td></td><td>6. Lưu các thông tin có gửi; chuỗi rỗng xóa số điện thoại hoặc địa chỉ.</td></tr>
<tr><td>7. Xem hồ sơ mới.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Thông tin không hợp lệ tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin chưa hợp lệ và không lưu.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
