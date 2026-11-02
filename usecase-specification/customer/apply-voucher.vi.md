# Áp dụng voucher

<table>
<tr><td colspan="2"><b>Tên use case:</b> Áp dụng voucher</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer kiểm tra một mã voucher còn dùng được trước khi đặt hàng để xem mức giảm. Bước này không trừ lượt dùng; lượt chỉ bị trừ khi đặt hàng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có gì thay đổi. Lượt dùng chỉ bị trừ khi đặt hàng.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Nhập mã voucher ở màn hình đặt hàng.</td><td></td></tr>
<tr><td>2. Gửi yêu cầu kiểm tra mã.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra mã tồn tại (không phân biệt hoa thường), còn trong thời hạn và còn lượt dùng.</td></tr>
<tr><td></td><td>4. Trả về phần trăm giảm của voucher.</td></tr>
<tr><td>5. Xem mức giảm dự kiến.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Mã không tồn tại, ngoài thời hạn hoặc đã hết lượt tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo voucher không dùng được.</td></tr>
<tr><td>3.1.2. Xem thông báo, sửa thông tin và quay lại bước 1.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
