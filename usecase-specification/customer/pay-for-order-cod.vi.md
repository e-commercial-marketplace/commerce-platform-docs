# Thanh toán khi nhận hàng (COD)

<table>
<tr><td colspan="2"><b>Tên use case:</b> Thanh toán khi nhận hàng (COD)</td></tr>
<tr><td colspan="2"><b>Mô tả ngắn:</b> Hệ thống cho phép Customer chọn thanh toán khi nhận hàng cho đơn hàng của mình.</td></tr>
<tr><td colspan="2"><b>Tác nhân chính:</b> Customer</td></tr>
<tr><td colspan="2"><b>Tác nhân phụ:</b> Không có.</td></tr>
<tr><td colspan="2"><b>Tiền điều kiện:</b><ul><li>Customer đã đặt đơn hàng COD và đơn hàng chưa thu tiền.</li></ul></td></tr>
<tr><td colspan="2"><b>Hậu điều kiện:</b><ul><li>Khoản tiền COD được hoàn tất sau khi mọi đợt giao hàng bắt buộc đều đã giao.</li></ul></td></tr>
<tr><td colspan="2"><b>Luồng chính:</b></td></tr>
<tr><th>Tác nhân</th><th>Hệ thống</th></tr>
<tr><td>1. Nhận kiện hàng và trả tiền COD khi nhận.</td><td></td></tr>
<tr><td></td><td>2. Ghi nhận kết quả giao hàng của kiện hàng.</td></tr>
<tr><td></td><td>3. Kiểm tra các đợt giao hàng bắt buộc của đơn hàng.</td></tr>
<tr><td></td><td>4. Hoàn tất khoản thanh toán COD chỉ khi tất cả các đợt giao hàng bắt buộc đều đã giao.</td></tr>
<tr><td></td><td>5. Cập nhật trạng thái đơn hàng và khoản thanh toán theo kết quả giao hàng.</td></tr>
<tr><td></td><td>6. Hiển thị trạng thái thanh toán.</td></tr>
<tr><td></td><td>7. Hiển thị kết quả.</td></tr>
<tr><td>8. Xem kết quả.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng thay thế:</b></td></tr>
<tr><td colspan="2"><b>Thay thế 1: Vẫn còn đợt giao hàng bắt buộc chưa được giao tại bước 3.</b></td></tr>
<tr><td></td><td>3.1. Giữ khoản thanh toán ở trạng thái chờ thu.</td></tr>
<tr><td>3.2. Tiếp tục nhận các đợt giao hàng còn lại.</td><td></td></tr>
<tr><td colspan="2"><b>Luồng ngoại lệ:</b></td></tr>
<tr><td colspan="2"><b>Ngoại lệ 1: Hệ thống gặp sự cố khi ghi nhận kết quả giao hàng tại bước 2.</b></td></tr>
<tr><td></td><td>2.1.1. Thông báo không thể ghi nhận kết quả giao hàng và kết thúc lần xử lý này.</td></tr>
<tr><td>2.1.2. Xem thông báo lỗi.</td><td></td></tr>
</table>
