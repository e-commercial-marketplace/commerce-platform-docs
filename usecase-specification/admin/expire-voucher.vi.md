# Cho voucher hết hạn

<table>
<tr><td colspan="2"><b>Tên use case:</b> Cho voucher hết hạn</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin cho voucher hết hạn ngay. Voucher không bị xóa để giữ lịch sử các đơn đã dùng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Thời hạn kết thúc của voucher là hiện tại và mã không dùng được nữa.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn cho một voucher hết hạn.</td><td></td></tr>
<tr><td>2. Xác nhận.</td><td></td></tr>
<tr><td></td><td>3. Kiểm tra voucher tồn tại.</td></tr>
<tr><td></td><td>4. Đặt thời hạn kết thúc là hiện tại; voucher chưa bắt đầu thì lùi ngày bắt đầu.</td></tr>
<tr><td>5. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Voucher không tồn tại tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không tìm thấy.</td></tr>
<tr><td>3.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
