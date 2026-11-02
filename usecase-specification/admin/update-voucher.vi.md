# Sửa voucher

<table>
<tr><td colspan="2"><b>Tên use case:</b> Sửa voucher</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Admin sửa mô tả, phần trăm giảm, thời hạn và số lượt. Mã không đổi được; phần trăm không đổi được sau khi voucher đã dùng; số lượt không được nhỏ hơn số đã dùng.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Admin</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Admin đã đăng nhập.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Voucher được cập nhật.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Chọn một voucher để sửa.</td><td></td></tr>
<tr><td></td><td>2. Hiển thị thông tin hiện tại.</td></tr>
<tr><td>3. Sửa thông tin cần đổi.</td><td></td></tr>
<tr><td>4. Gửi yêu cầu cập nhật.</td><td></td></tr>
<tr><td></td><td>5. Kiểm tra voucher tồn tại và các thay đổi được phép.</td></tr>
<tr><td></td><td>6. Lưu thay đổi và trả về kết quả.</td></tr>
<tr><td>7. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Đổi phần trăm của voucher đã dùng, thời hạn sai, hoặc số lượt nhỏ hơn số đã dùng tại bước 5.</b></td></tr>
<tr><td></td><td>5.1.1. Thông báo không lưu được.</td></tr>
<tr><td>5.1.2. Xem thông báo, sửa thông tin và quay lại bước 3.</td><td></td></tr>
<tr><td colspan="2"><b>Thay thế 2: Voucher không tồn tại tại bước 5.</b></td></tr>
<tr><td></td><td>5.2.1. Thông báo không tìm thấy.</td></tr>
<tr><td>5.2.2. Xem thông báo.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi xử lý yêu cầu tại bước 4.</b></td></tr>
<tr><td></td><td>4.1.1. Thông báo không thể hoàn tất yêu cầu do lỗi hệ thống và kết thúc lần xử lý này.</td></tr>
<tr><td>4.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
