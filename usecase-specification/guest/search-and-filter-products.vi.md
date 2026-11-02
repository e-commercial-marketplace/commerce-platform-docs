# Tìm kiếm và lọc sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tìm kiếm và lọc sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Guest tìm sản phẩm đang bán theo từ khóa, danh mục, shop, khoảng giá, điểm đánh giá và cách sắp xếp. Khi không sản phẩm nào chứa từ khóa, hệ thống tìm các sản phẩm gần nghĩa.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng tìm kiếm.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị ô tìm kiếm và các bộ lọc.</td></tr>
<tr><td>3. Nhập từ khóa hoặc chọn bộ lọc và cách sắp xếp.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu tìm kiếm.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra giá tối thiểu không lớn hơn giá tối đa.</td></tr>
<tr><td></td><td>6. Tìm sản phẩm đang bán có tên hoặc mô tả chứa từ khóa và thỏa các bộ lọc.</td></tr>
<tr><td></td><td>7. Nếu không có kết quả, tìm các sản phẩm gần nghĩa nhất và vẫn áp các bộ lọc.</td></tr>
<tr><td></td><td>8. Trả về trang kết quả.</td></tr>
<tr><td>9. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Giá tối thiểu lớn hơn giá tối đa tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo bộ lọc giá chưa hợp lệ.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Vẫn không có kết quả tại bước 8.</b></td></tr>
<tr><td></td><td>8.1.1. Hiển thị danh sách rỗng.</td></tr>
<tr><td>8.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
