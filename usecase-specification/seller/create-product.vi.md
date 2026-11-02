# Tạo sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tạo sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Seller tạo sản phẩm. Sản phẩm mặc định đang bán và khi đó bắt buộc có ảnh. Ảnh được đưa lên bằng use case Tải ảnh lên.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Seller</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Seller đã đăng nhập và đã được duyệt.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Sản phẩm được tạo và thuộc về Seller trong token.</li><li>Sản phẩm tìm được theo nghĩa sau tối đa 10 phút (job đồng bộ embedding).</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở biểu mẫu tạo sản phẩm.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị biểu mẫu và danh mục.</td></tr>
<tr><td>3. Nhập tên, mô tả, giá, tồn kho, chọn danh mục và chọn ảnh.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu tạo.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra giá lớn hơn 0, tồn kho không âm, danh mục tồn tại, sản phẩm đang bán có ảnh và ảnh thuộc Cloudinary của hệ thống.</td></tr>
<tr><td></td><td>6. Lưu sản phẩm và trả về kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Thông tin không hợp lệ tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo thông tin chưa hợp lệ và không tạo sản phẩm.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
