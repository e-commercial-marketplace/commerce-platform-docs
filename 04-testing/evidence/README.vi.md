# Bằng chứng Kiểm thử

Thư mục này lưu bằng chứng thu thập thủ công cho bộ automated test của frontend.

Chưa có ảnh chụp nào được commit. Các file được nêu trong kế hoạch dưới đây hiện **chưa** tồn tại.

## Bằng chứng cần có

| Bằng chứng | Tên file gợi ý | Thu thập từ |
|---|---|---|
| Jest PASS summary | `jest-pass-<commit>.png` | Output của `npm test` trong `commerce-platform-frontend-mobile` |
| Coverage summary | `jest-coverage-<commit>.png` | Output của `npm run test:coverage` |
| Typecheck pass | `typecheck-pass-<commit>.png` | Output của `npm run typecheck` |
| Hành vi app tùy chọn | `app-cart-empty-<commit>.png` | App đang chạy, trạng thái cart rỗng |

## Quy tắc

- Ghi nhãn mỗi mục thu thập kèm commit hash của frontend và ngày chụp.
- Chỉ commit ảnh chụp thật. Không bao giờ commit file giữ chỗ như thể đó là kết quả thật.
- Bằng chứng mang tính tại một thời điểm và gắn với frontend revision ở mục 6 của
  [`../README.vi.md`](../README.vi.md).
- Nếu frontend revision thay đổi, bằng chứng cũ chỉ còn giá trị cho commit mà nó đã ghi lại.
