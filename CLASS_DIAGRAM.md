# Domain Class Diagram

## 1. Full Mermaid view

This diagram is the Mermaid documentation view of
[`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml). The
PlantUML file remains the source reference for the complete class model.

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

## 2. Important interpretation rules

- `sellerStatus` applies only when `User.role = SELLER`.
- `OrderItemStatus` describes seller processing; `ShipmentStatus` describes delivery progress.
- `OrderStatus` is an order-level aggregate; seller-specific `FAILED` and `RETURNED` outcomes belong
  to `ShipmentStatus`.
- `OrderItem.productName` and `OrderItem.unitPrice` preserve checkout history.
- A shipment contains order items from exactly one seller.
- A shipment contains order items from exactly one order; `Shipment.order == OrderItem.order` for
  every mapped item. In the MVP, mappings use whole order-item lines and one `OrderItem` belongs to at
  most one `Shipment`.
- `FAILED` does not restore stock. `RETURNED` means the package reached the seller warehouse and is
  the only shipment transition that restores its shipped quantity.
- Prepayment orders expire 24 hours after creation when payment is incomplete; COD is collected only
  when all required non-cancelled shipments are delivered. Mixed `FAILED`/`RETURNED` outcomes have no
  partial COD settlement in the MVP.
- Mixed terminal outcomes remain seller-scoped on `ShipmentStatus`; the model does not invent an
  order-level failure enum.
- Direct item cancellation is allowed before shipment assignment; assigned unshipped lines use the
  atomic shipment-cancellation path, while `SHIPPED` or later lines cannot be cancelled.
- Before payment completion, cancellation recalculates the current order payable total and pending
  payment amount from remaining line snapshots; all-cancelled orders become `CANCELLED`.
- Guest AI product search is stateless/public; persisted `AiChatSession` data belongs to `CUSTOMER`.
- One order may have multiple payment attempts so failed VNPAY attempts can be retried.
- Human chat and AI chat are separate concepts.
- AI is read-only for Order, Payment, and Shipment in the initial design.

## 3. Feature-level diagrams

Use the focused diagrams for implementation work:

- [`features/auth/README.md`](features/auth/README.md)
- [`features/user/README.md`](features/user/README.md)
- [`features/catalog/README.md`](features/catalog/README.md)
- [`features/cart/README.md`](features/cart/README.md)
- [`features/order/README.md`](features/order/README.md)
- [`features/payment/README.md`](features/payment/README.md)
- [`features/shipment/README.md`](features/shipment/README.md)
- [`features/voucher/README.md`](features/voucher/README.md)
- [`features/review/README.md`](features/review/README.md)
- [`features/chat/README.md`](features/chat/README.md)
- [`features/ai/README.md`](features/ai/README.md)
- [`features/common/README.md`](features/common/README.md)
