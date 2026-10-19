# Tìm kiếm sản phẩm bằng trợ lý AI

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tìm kiếm sản phẩm bằng trợ lý AI</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Guest tìm kiếm sản phẩm bằng trợ lý AI.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Dữ liệu marketplace không thay đổi khi được tra cứu.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở chức năng trợ lý AI.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị nơi nhập câu hỏi.</td></tr>
<tr><td>3. Nhập câu hỏi về sản phẩm công khai.</td><td></td></tr>
<tr><td>4. Gửi câu hỏi.</td><td></td></tr>
<tr><td></td><td>5. Tìm thông tin phù hợp và tạo câu trả lời.</td></tr>
<tr><td></td><td>6. Hiển thị câu trả lời.</td></tr>
<tr><td>7. Xem câu trả lời.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Trợ lý chưa thể trả lời hoặc câu hỏi ngoài phạm vi thông tin công khai.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo hiện chưa thể trả lời câu hỏi.</td></tr>
<tr><td>4.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
