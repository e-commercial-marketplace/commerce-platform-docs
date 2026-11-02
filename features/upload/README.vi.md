# Tính năng Upload ảnh

## Mục đích

Cho người bán và người tham gia trò chuyện đính kèm ảnh thật mà file không phải đi qua backend.

## Trách nhiệm

- Cấp chữ ký upload Cloudinary có thời hạn ngắn cho người được phép upload.
- Từ chối đường dẫn ảnh sản phẩm và ảnh chat không thuộc tài khoản Cloudinary của dự án.

## API

```text
POST /api/v1/uploads/signature
```

Chi tiết: [`API_CONTRACT.md`](API_CONTRACT.md).

## Quy tắc quan trọng

- API secret chỉ nằm ở backend; client chỉ nhận chữ ký.
- Backend tự chọn `public_id`, nên client không ghi đè được ảnh đã có.
- Không thêm bảng: sản phẩm và tin nhắn vẫn lưu `image_url`.
- Biến `CLOUDINARY_*` để trống thì tắt tính năng: API ký trả `503` và không kiểm tra URL.
