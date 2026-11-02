# Duyệt catalog

<table>
<tr><td colspan="2"><b>Tên use case:</b> Duyệt catalog</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer duyệt sản phẩm đang bán theo trang và danh mục. Không cần token.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở catalog.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị danh mục và trang sản phẩm đang bán, mặc định điểm đánh giá cao trước.</td></tr>
<tr><td>3. Chọn danh mục hoặc chuyển trang.</td><td></td></tr>
<tr><td></td><td>4. Hiển thị trang sản phẩm tương ứng.</td></tr>
<tr><td>5. Xem danh sách sản phẩm.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Không có sản phẩm phù hợp tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Hiển thị danh sách rỗng.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
