# Tìm sản phẩm bằng trợ lý AI

<table>
<tr><td colspan="2"><b>Tên use case:</b> Tìm sản phẩm bằng trợ lý AI</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Guest hỏi trợ lý AI về sản phẩm và chính sách. Guest không có lịch sử, không có gì được lưu.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Guest</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Người dùng chưa đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Không có.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Mở trợ lý AI.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị nơi nhập câu hỏi.</td></tr>
<tr><td>3. Nhập câu hỏi.</td><td></td></tr>
<tr><td>4. Gửi câu hỏi.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra hạn mức câu hỏi trong một phút.</td></tr>
<tr><td></td><td>6. Dùng LLM và các công cụ chỉ đọc (danh mục, tìm sản phẩm, chính sách) để tạo câu trả lời từ dữ liệu thật.</td></tr>
<tr><td></td><td>7. Nếu không gọi được LLM (thiếu key, lỗi, đang chờ sau 429, hoặc quá 20 câu một phút), trả lời bằng luật; nếu đã tắt trả lời bằng luật thì báo lỗi.</td></tr>
<tr><td></td><td>8. Trả về câu trả lời kèm các sản phẩm tìm được.</td></tr>
<tr><td>9. Xem câu trả lời và thẻ sản phẩm.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Vượt quá 60 câu hỏi trong một phút tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo đã vượt giới hạn và yêu cầu thử lại sau.</td></tr>
<tr><td>5.1.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
