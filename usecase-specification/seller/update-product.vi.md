# Sửa sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Sửa sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller sửa thông tin hoặc bán lại sản phẩm của chính mình. Chỉ các trường có gửi mới đổi.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Sản phẩm được cập nhật và cache chi tiết bị xóa.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở một sản phẩm của tôi để sửa.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị thông tin hiện tại.</td></tr>
<tr><td>3. Sửa thông tin cần đổi.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra sản phẩm thuộc về Seller, danh mục tồn tại và sản phẩm đang bán vẫn có ảnh.</td></tr>
<tr><td></td><td>6. Lưu thay đổi và trả về kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Thông tin không hợp lệ tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin chưa hợp lệ và không lưu.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Sản phẩm không tồn tại hoặc của Seller khác tại bước 5.</b></td></tr>
<tr><td></td><td>5.2.1. Thông báo không sửa được sản phẩm.</td></tr>
<tr><td>5.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
