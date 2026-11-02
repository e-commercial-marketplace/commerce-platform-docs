# Xem lịch sử trò chuyện AI

<table>
<tr><td colspan="2"><b>Tên use case:</b> Xem lịch sử trò chuyện AI</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer xem các phiên trò chuyện với trợ lý AI và toàn bộ tin nhắn của từng phiên. Guest không có lịch sử.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở lịch sử trò chuyện AI.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị các phiên của Customer, mới nhất trước.</td></tr>
<tr><td>3. Chọn một phiên.</td><td></td></tr>
<tr><td></td><td>4. Kiểm tra phiên thuộc về Customer và trả về tin nhắn, cũ trước.</td></tr>
<tr><td>5. Xem các tin nhắn.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Phiên không tồn tại hoặc của người khác tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không tìm thấy phiên.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 3.</b></td></tr>
<tr><td></td><td>3.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>3.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
