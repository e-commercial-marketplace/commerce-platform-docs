# Sơ đồ class miền

## 1. Khung nhìn Mermaid đầy đủ

Sơ đồ này là Mermaid documentation view của
[`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml). File
PlantUML vẫn là source reference cho complete class model.

```mermaid
classDiagram
    class User {
        UUID id
        String name
        String email
        String passwordHash
        String phone
        String address
        Role role
        SellerStatus sellerStatus?
    }

    class Category {
        UUID id
        String name
    }

    class Product {
        UUID id
        String name
        String description
        String imageUrl
        Decimal price
        int stock
        Decimal ratingAverage
        int reviewCount
        boolean active
    }

    class CartItem {
        UUID id
        int quantity
        DateTime createdAt
    }

    class Order {
        UUID id
        OrderStatus status
        DateTime createdAt
        String shippingAddress
        Decimal shippingFee
        Decimal discountAmount
        Decimal totalAmount
    }

    class OrderItem {
        UUID id
        String productName
        Decimal unitPrice
        int quantity
        OrderItemStatus status
        String cancelReason?
    }

    class Payment {
        UUID id
        Decimal amount
        PaymentMethod method
        PaymentStatus status
        String transactionId?
        DateTime paidAt?
    }

    class Voucher {
        String code
        String description
        int discountPercent
        DateTime validFrom
        DateTime validTo
        int usageLimit
        int timesUsed
    }

    class Shipment {
        UUID id
        DeliveryMethod deliveryMethod
        ShipmentStatus status
        String trackingCode?
        Decimal shippingFee
        DateTime shippedAt?
        DateTime deliveredAt?
    }

    class Review {
        UUID id
        int rating
        String comment
        DateTime createdAt
    }

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

    class AiChatSession {
        UUID id
        DateTime createdAt
    }

    class AiMessage {
        UUID id
        AiRole role
        String content
        Decimal confidence?
        DateTime createdAt
    }

    class Role {
        <<enumeration>>
        CUSTOMER
        SELLER
        ADMIN
    }

    class SellerStatus {
        <<enumeration>>
        PENDING
        APPROVED
        REJECTED
        SUSPENDED
    }

    class OrderStatus {
        <<enumeration>>
        PENDING
        PAID
        DELIVERED
        CANCELLED
    }

    class OrderItemStatus {
        <<enumeration>>
        PENDING
        CONFIRMED
        CANCELLED
    }

    class PaymentMethod {
        <<enumeration>>
        COD
        VNPAY
    }

    class PaymentStatus {
        <<enumeration>>
        PENDING
        COMPLETED
        FAILED
        REFUNDED
    }

    class DeliveryMethod {
        <<enumeration>>
        PLATFORM_DELIVERY
        SELF_DELIVERY
    }

    class ShipmentStatus {
        <<enumeration>>
        PENDING
        PREPARING
        SHIPPED
        DELIVERED
        FAILED
        RETURNED
        CANCELLED
    }

    class AiRole {
        <<enumeration>>
        USER
        ASSISTANT
        SYSTEM
    }

    User "1" --> "0..*" CartItem : owns
    CartItem "0..*" --> "1" Product : product
    User "1" --> "0..*" Order : places
    Order "1" *-- "1..*" OrderItem : items
    OrderItem "0..*" --> "1" Product : product
    OrderItem "0..*" --> "1" User : seller
    User "1" --> "0..*" Product : sells
    Category "1" --> "0..*" Product : contains
    Order "1" --> "0..*" Payment : payment_attempts
    Order "0..*" --> "0..1" Voucher : applies
    Order "1" *-- "0..*" Shipment : shipments
    Shipment "0..1" --> "1..*" OrderItem : contains
    Shipment "0..*" --> "1" User : seller
    User "1" --> "0..*" Review : writes
    Review "0..*" --> "1" Product : reviews
    User "1" --> "0..*" Conversation : customer
    User "1" --> "0..*" Conversation : seller
    Conversation "1" *-- "0..*" Message : messages
    Message "0..*" --> "1" User : sender
    Conversation "0..*" --> "0..1" Product : about_product
    Conversation "0..*" --> "0..1" Order : about_order
    User "1" --> "0..*" AiChatSession : owns
    AiChatSession "1" *-- "0..*" AiMessage : turns
```

## 2. Quy tắc diễn giải quan trọng

- `sellerStatus` chỉ áp dụng khi `User.role = SELLER`.
- `OrderItemStatus` mô tả seller processing; `ShipmentStatus` mô tả delivery progress.
- `OrderStatus` là order-level aggregate; các `FAILED` và `RETURNED` outcomes seller-specific thuộc
  về `ShipmentStatus`.
- `OrderItem.productName` và `OrderItem.unitPrice` bảo lưu checkout history.
- Một shipment chứa order items từ đúng một seller.
- Một shipment chứa order items từ đúng một order; `Shipment.order == OrderItem.order` cho
  mọi mapped item. Trong MVP, mappings dùng whole order-item lines và một `OrderItem` thuộc về tối
  đa một `Shipment`.
- `FAILED` không khôi phục stock. `RETURNED` nghĩa là package đã tới seller warehouse và là
  shipment transition duy nhất khôi phục shipped quantity của nó.
- Prepayment orders hết hạn 24 giờ sau khi tạo khi payment chưa complete; COD chỉ được collected
  khi mọi required non-cancelled shipments đã delivered. Mixed `FAILED`/`RETURNED` outcomes không có
  partial COD settlement trong MVP.
- Mixed terminal outcomes vẫn seller-scoped trên `ShipmentStatus`; model không invent
  order-level failure enum.
- Direct item cancellation được phép trước shipment assignment; các assigned unshipped lines dùng
  atomic shipment-cancellation path, trong khi các lines `SHIPPED` hoặc sau đó không thể cancelled.
- Trước payment completion, cancellation tính lại current order payable total và pending
  payment amount từ remaining line snapshots; các all-cancelled orders trở thành `CANCELLED`.
- Guest AI product search stateless/public; persisted `AiChatSession` data thuộc về `CUSTOMER`.
- Một order có thể có nhiều payment attempts để failed VNPAY attempts có thể retried.
- Human chat và AI chat là các khái niệm tách biệt.
- AI read-only đối với Order, Payment và Shipment trong thiết kế ban đầu.

## 3. Sơ đồ cấp feature

Dùng các focused diagrams cho công việc implementation:

- [`features/auth/README.vi.md`](features/auth/README.vi.md)
- [`features/user/README.vi.md`](features/user/README.vi.md)
- [`features/catalog/README.vi.md`](features/catalog/README.vi.md)
- [`features/cart/README.vi.md`](features/cart/README.vi.md)
- [`features/order/README.vi.md`](features/order/README.vi.md)
- [`features/payment/README.vi.md`](features/payment/README.vi.md)
- [`features/shipment/README.vi.md`](features/shipment/README.vi.md)
- [`features/voucher/README.vi.md`](features/voucher/README.vi.md)
- [`features/review/README.vi.md`](features/review/README.vi.md)
- [`features/chat/README.vi.md`](features/chat/README.vi.md)
- [`features/ai/README.vi.md`](features/ai/README.vi.md)
- [`features/common/README.vi.md`](features/common/README.vi.md)
