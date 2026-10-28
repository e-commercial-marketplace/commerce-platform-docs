# Tính năng Human Chat

## Mục đích

Sở hữu hội thoại customer/seller, tin nhắn lưu trữ, cập nhật thời gian thực và lịch sử tin nhắn.

## Trách nhiệm

- Ghép cặp customer và seller trong một hội thoại.
- Lưu trữ tin nhắn văn bản/ảnh và trạng thái đã đọc.
- Cung cấp cập nhật thời gian thực qua WebSocket.
- Cung cấp tải lịch sử offline.
- Áp dụng xử lý retry và idempotency cho tin nhắn.
- Cho phép hội thoại tham chiếu sản phẩm hoặc đơn hàng.

## API

```text
WS    /ws/chat
GET   /api/v1/conversations
POST  /api/v1/conversations
GET   /api/v1/conversations/{id}/messages
POST  /api/v1/conversations/{id}/messages
PATCH /api/v1/conversations/{id}/read
```

Chi tiết: [`API_CONTRACT.md`](API_CONTRACT.md).

## Phân quyền

| Tác nhân | Thao tác cho phép | Ràng buộc |
|---|---|---|
| CUSTOMER | Dùng các hội thoại mà họ là participant customer | Không được truy cập hội thoại của customer khác. |
| SELLER | Dùng các hội thoại mà họ là participant seller | Không được truy cập hội thoại của seller khác. |
| ADMIN | Giám sát vận hành theo quy trình quản trị | Giám sát quản trị không phải mạo danh participant. |
| Trợ lý AI | Không sở hữu human-chat | AI chat là tính năng và mô hình lưu trữ riêng. |

## Sơ đồ lớp trọng tâm

```mermaid
classDiagram
    class Conversation {
        UUID id
        DateTime createdAt
        DateTime lastMessageAt?
    }

    class Message {
        UUID id
        String content
        String imageUrl?
        boolean isRead
        DateTime createdAt
    }

    class User {
        UUID id
        Role role
    }

    class Product {
        UUID id
        String name
    }

    class Order {
        UUID id
        OrderStatus status
    }

    User "1" --> "0..*" Conversation : customer
    User "1" --> "0..*" Conversation : seller
    Conversation "1" *-- "0..*" Message : messages
    Message "0..*" --> "1" User : sender
    Conversation "0..*" --> "0..1" Product : about
    Conversation "0..*" --> "0..1" Order : about
```

## Quy tắc quan trọng

- Quy tắc hội thoại MVP là `(customer, seller)` duy nhất; mở hội thoại là get-or-create.
- Customer mở hội thoại với seller đã `APPROVED`; seller mở hội thoại với customer. `orderId` tùy chọn phải
  là đơn của customer đó.
- Người ngoài hội thoại nhận `404 RESOURCE_NOT_FOUND`, để không lộ id hội thoại.
- Tin nhắn được gửi qua REST kèm header `Idempotency-Key` bắt buộc; gửi lại cùng key trả về tin đã lưu thay
  vì tạo bản trùng (`FR-CHAT-04`).
- Human chat và AI chat là hai khái niệm riêng biệt.

## Cập nhật thời gian thực

- `WS /ws/chat` là kênh chỉ đẩy. Handshake mang `Authorization: Bearer <access-token>` và chỉ `CUSTOMER`,
  `SELLER` được kết nối. Client không gửi gì; server đẩy sự kiện JSON sau khi transaction database commit:
  - `MESSAGE_CREATED` kèm tin mới, tới các kết nối đang mở của cả hai participant;
  - `MESSAGES_READ` kèm `conversationId` và `readerId`, tới participant còn lại.
- REST và database vẫn là nguồn sự thật: client bị offline tải lại lịch sử bằng
  `GET /conversations/{id}/messages`.
- Shortcut: các kết nối đang mở được giữ trong RAM, đủ cho một instance backend. Chạy nhiều instance cần
  một broker dùng chung (ví dụ Redis pub/sub).

## Mô hình dữ liệu

Human chat có một hàng hội thoại participant và các hàng tin nhắn con. Mô hình này tách biệt với mô hình session/message của AI.

### Quan hệ bảng

- `users (1) -> conversations (many)` xuất hiện hai lần: một lần qua `customerId` và một lần qua `sellerId`.
- `conversations (1) -> messages (many)`: xóa hay lưu trữ hội thoại phải giữ nguyên chính sách lưu giữ tin nhắn đã chọn.
- `messages.senderId -> users.id` xác định người gửi thực tế và phải được kiểm tra với tư cách thành viên hội thoại.
- `conversations.productId` và `conversations.orderId` là các tham chiếu tùy chọn dùng làm ngữ cảnh.

### Ý nghĩa trường

| Bảng | Trường | Ý nghĩa |
|---|---|---|
| `conversations` | `id` | Định danh hội thoại. |
| `conversations` | `createdAt` | Thời điểm hội thoại customer/seller được mở. |
| `conversations` | `lastMessageAt` | Timestamp tùy chọn dùng để sắp xếp; rỗng khi chưa có tin nhắn. |
| `conversations` | `customerId` | Participant customer. |
| `conversations` | `sellerId` | Participant seller. |
| `conversations` | `productId` | Ngữ cảnh sản phẩm tùy chọn cho hội thoại. |
| `conversations` | `orderId` | Ngữ cảnh đơn hàng tùy chọn cho hỗ trợ hoặc thảo luận sau bán. |
| `messages` | `id` | Định danh tin nhắn. |
| `messages` | `content` | Nội dung văn bản của tin nhắn. |
| `messages` | `imageUrl` | Tham chiếu ảnh đính kèm tùy chọn. |
| `messages` | `isRead` | Người nhận dự kiến đã đọc tin nhắn hay chưa. |
| `messages` | `createdAt` | Thời điểm tạo tin nhắn dùng để sắp xếp. |
| `messages` | `conversationId` | Hội thoại cha. |
| `messages` | `senderId` | Người dùng đã gửi tin nhắn. |
| `messages` | `clientMessageId` | `Idempotency-Key` của người gửi; duy nhất theo từng người gửi. |

Migration V9 lịch sử là placeholder. V13 tạo bảng conversation/message; V19 thêm cột
`messages.client_message_id` với ràng buộc duy nhất `(sender_id, client_message_id)`.
