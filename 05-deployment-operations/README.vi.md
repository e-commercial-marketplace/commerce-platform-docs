# Triển khai, vận hành và kiểm tra bảo mật

## Phạm vi và baseline

Hướng dẫn/checklist cho backend `2f34fef` ngày 2026-10-28; không khẳng định đã triển khai,
backup, restore hay nghiệm thu. Các lệnh local và cấu hình chi tiết nằm ở `backend/README.md`,
`backend/docker-compose.yml` và `backend/e2e/README.md`. Xem [kết quả test backend](../04-testing/BACKEND_VERIFICATION.vi.md).

## Local và cấu hình

1. Chuẩn bị Java 21, Maven, Docker Compose; sao chép `backend/.env.example` thành `.env` khi cần.
   Không commit `.env`, JWT secret, mật khẩu, mã reset, khóa VNPAY hay AI.
2. Trong `backend`, dùng `docker compose up -d --build`; xem `docker compose ps` và
   `docker compose logs backend`, kiểm tra `GET http://localhost:8080/actuator/health`.
3. Compose ghi đè `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` và `REDIS_URL` cho container backend:
   container dùng Postgres/Redis của Compose, **không dùng managed DB từ `.env`**. API chạy trên host
   bằng `mvn spring-boot:run` dùng cấu hình host; Redis Compose không mở cổng ra host.
4. Dừng bằng `docker compose down` để giữ volume; `docker compose down -v` **xóa dữ liệu**,
   chỉ dùng nếu đã được phép và hiểu rõ phạm vi.

## Checklist release / Render

- [ ] Ghi commit artifact, môi trường, người thực hiện và kế hoạch quay lui; kiểm tra test/CI tương ứng.
- [ ] Xác nhận database, backup và quyền khôi phục; không mặc định backup/restore đã thành công.
- [ ] Cấu hình `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`, `APP_JWT_SECRET` qua secret manager; cấu hình
  CORS theo origin tin cậy và email Brevo (`BREVO_API_KEY`, gửi qua HTTPS vì Render miễn phí chặn cổng SMTP), VNPAY, AI, Redis khi cần. Không dùng credential local ở production.
- [ ] Kiểm tra Flyway migration mới và tương thích với phiên bản cũ; không sửa migration đã áp dụng.
  Migration chạy khi backend khởi động: kiểm tra log/health trước khi đưa traffic.
- [ ] Sau release, kiểm tra health, GET catalog công khai, login tài khoản kiểm thử được phép và một
  GET có xác thực chỉ đọc. Không ghi token hoặc dữ liệu nhạy cảm vào evidence.
- [ ] Không chạy collection E2E tạo user/order trên Render. Dừng release khi smoke test thất bại.

Rollback: xác định app cũ có tương thích schema hiện tại trước khi đưa artifact cũ trở lại. Không tự
động đảo migration hoặc restore database production; cần kế hoạch dữ liệu, phê duyệt và đánh giá mất dữ liệu.
Ghi lại sự cố, thời gian và kết quả; thử backup/restore riêng ở môi trường an toàn trước khi coi là khả dụng.

## Checklist bảo mật và sự cố

| Kiểm tra | Bằng chứng hiện có / việc còn lại |
|---|---|
| Token sai/hết hạn, refresh rotation/logout | `AuthIntegrationTest`, `AuthTokenEdgeCaseTest`; test H2 không thay thế kiểm tra môi trường deploy. |
| Role và ownership order/seller/AI | `OrderApiTest`, `SellerProductApiTest`, `ShipmentApiTest`, `AiApiTest`; xác định và ghi test case cụ thể cho từng yêu cầu trước nghiệm thu. |
| Chữ ký, callback lặp VNPAY | `VnpayGatewayTest`, `PaymentApiTest`; chưa xác minh giao dịch sandbox thật trong lần này. |
| CORS, secret, log | `CorsConfigTest` có test cấu hình; rà lại origin, secret và log thực tế trước release. |
| Redis lỗi / AI provider lỗi | Có test fallback ở `CartCacheApiTest`, `AiApiTest`; theo dõi log và xác minh môi trường thực tế. Redis không thuộc health check. |

Tham khảo [security flow](../features/auth/SECURITY_FLOW.vi.md). Khi provider/DB lỗi, kiểm tra health,
log đã che thông tin nhạy cảm và cấu hình; không tắt xác thực để xử lý sự cố. Nếu payment cần đối soát
hoặc hoàn tiền, chuyển xử lý theo quy trình được phê duyệt thay vì tự sửa dữ liệu.
