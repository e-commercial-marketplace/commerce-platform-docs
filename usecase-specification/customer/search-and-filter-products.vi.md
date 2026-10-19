# Tìm kiếm và lọc sản phẩm

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tìm kiếm và lọc sản phẩm</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer tìm kiếm và lọc sản phẩm phù hợp.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Dữ liệu sản phẩm không thay đổi khi được tìm kiếm.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng tìm kiếm và lọc sản phẩm.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị sản phẩm phù hợp tác nhân được phép xem.</td></tr>
<tr><td>3. Nhập từ khóa hoặc chọn điều kiện lọc.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu tìm kiếm.</td><td></td></tr>
<tr><td></td><td>5. Tìm sản phẩm phù hợp.</td></tr>
<tr><td></td><td>6. Hiển thị kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Không có kết quả phù hợp tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo chưa tìm thấy dữ liệu phù hợp.</td></tr>
<tr><td>5.1.2. Thay đổi điều kiện tìm kiếm và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
