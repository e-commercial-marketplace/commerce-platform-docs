# Xem voucher

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xem voucher</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho mọi người dùng, kể cả Guest, xem các voucher đang dùng được và sao chép mã. Dùng voucher khi đặt hàng vẫn cần đăng nhập.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest, Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Không yêu cầu đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có gì thay đổi. Xem hoặc sao chép mã không trừ lượt dùng.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở trang chủ hoặc chọn xem tất cả voucher.</td><td></td></tr>
<tr><td></td><td>2. Lấy các voucher đang trong thời hạn và còn lượt dùng, phần trăm giảm cao nhất trước.</td></tr>
<tr><td></td><td>3. Trả về mã, mô tả, phần trăm giảm và hạn dùng của từng voucher.</td></tr>
<tr><td>4. Xem danh sách và chọn sao chép một mã.</td><td></td></tr>
<tr><td></td><td>5. Ứng dụng chép mã vào bộ nhớ tạm và báo đã sao chép.</td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Không có voucher nào đang dùng được tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Trả về danh sách rỗng và ứng dụng báo chưa có voucher.</td></tr>
<tr><td>2.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Guest muốn dùng voucher tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Nhắc đăng nhập để dùng voucher khi đặt hàng.</td></tr>
<tr><td>4.1.2. Chọn đăng nhập hoặc tiếp tục xem.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 1.</b></td></tr>
<tr><td></td><td>1.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>1.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
