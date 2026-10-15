# Hợp đồng API MVP

## 1. Mục đích và phạm vi

Tài liệu này định nghĩa hợp đồng request và response tối thiểu cho mobile
MVP đầu tiên hướng tới customer. Đây là ranh giới dùng chung giữa React Native client và Spring Boot backend.

Hợp đồng bao gồm:

- xác thực customer;
- public catalog và product search;
- quản lý cart;
- checkout và tạo order COD;
- tra cứu order của customer;
- product reviews cơ bản;
- AI product search cho guest/customer.

Seller/Admin screens, human chat, voucher application, tích hợp VNPAY, advanced shipment tracking,
product variants và review attachments được hoãn khỏi implementation đầu tiên của hợp đồng này.

## 2. Quy ước chung

### 2.1 Base path và authentication

Mọi endpoints dùng base path `/api`. Các protected requests gửi:

```http
Authorization: Bearer <access-token>
```

Các requests public catalog, guest AI product-search và review-list không yêu cầu token. Các thao tác cart,
checkout, order và tạo review yêu cầu `CUSTOMER` đã authenticated.

### 2.2 Định dạng dữ liệu

| Giá trị | Định dạng |
|---|---|
| Identifier | UUID string |
| Timestamp | Chuỗi UTC ISO-8601, ví dụ `2026-10-12T10:30:00Z` |
| Money | Chuỗi decimal tính bằng `VND`, ví dụ `"2800.00"`, để bảo toàn precision giữa Java và JavaScript |
| Page index | Số nguyên zero-based |
| Page size | Số nguyên dương; backend áp dụng maximum |
| Empty optional value | `null`, không phải chuỗi rỗng |

### 2.3 Wire envelope và error response

Mọi top-level endpoint response có body đều dùng `ApiResponse<T>` từ module `common`. Response
`204 No Content` không có body. Request body vẫn là JSON thuần. Các ví dụ response dưới đây bao gồm
đầy đủ wire envelope, trừ khi được ghi rõ là nested data payload. Kết quả riêng của endpoint luôn nằm
trong `data`.

Các successful responses dùng envelope này:

```json
{
  "success": true,
  "data": "<endpoint-specific payload>",
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Mọi rejected request dùng cùng shape:

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "fieldErrors": {
      "quantity": "Must be greater than zero"
    }
  },
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

`fieldErrors` là optional và có thể `null`. Backend không được expose framework binding details,
stack traces, provider secrets hoặc SQL errors.

Các error codes ban đầu:

```text
VALIDATION_ERROR
AUTHENTICATION_REQUIRED
FORBIDDEN
RESOURCE_NOT_FOUND
PRODUCT_INACTIVE
INSUFFICIENT_STOCK
EMPTY_CART
ORDER_NOT_PAYABLE
CART_ITEM_NOT_FOUND
CANNOT_CANCEL_AFTER_SHIPMENT
CANNOT_CANCEL_AFTER_PAYMENT
REVIEW_NOT_ELIGIBLE
REVIEW_ALREADY_EXISTS
VOUCHER_EXPIRED
VOUCHER_EXHAUSTED
VOUCHER_ALREADY_APPLIED
PAYMENT_FAILED
INTERNAL_ERROR
```

Ánh xạ HTTP cho MVP đầu tiên:

| Status | Use |
|---|---|
| `200 OK` | Đọc hoặc mutation thành công có trả về body |
| `201 Created` | Tạo resource thành công, như registration hoặc tạo order |
| `204 No Content` | Xóa thành công khi endpoint không trả về body |
| `400 Bad Request` | JSON sai định dạng, validation failure hoặc invalid state transition |
| `401 Unauthorized` | Thiếu, hết hạn hoặc invalid authentication |
| `403 Forbidden` | Actor đã authenticated thiếu role hoặc ownership bắt buộc |
| `404 Not Found` | Resource không tồn tại hoặc không visible với caller |
| `409 Conflict` | Business conflict như insufficient stock, duplicate review hoặc order không còn payable |
| `500 Internal Server Error` | Lỗi server không mong đợi được ánh xạ thành `INTERNAL_ERROR` |

## 3. Authentication

### Đăng ký

```http
POST /api/auth/register
Content-Type: application/json
```

```json
{
  "email": "customer@example.com",
  "password": "strong-password",
  "name": "Customer"
}
```

### Đăng nhập

```http
POST /api/auth/login
Content-Type: application/json
```

```json
{
  "email": "customer@example.com",
  "password": "strong-password"
}
```

### Auth response

```json
{
  "success": true,
  "data": {
    "accessToken": "jwt-access-token",
    "refreshToken": "jwt-refresh-token",
    "expiresIn": 900,
    "user": {
      "id": "user-uuid",
      "email": "customer@example.com",
      "name": "Customer",
      "role": "CUSTOMER"
    }
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Ba endpoint authentication trả về `ApiResponse<AuthResponse>` khi thành công. Credentials không hợp lệ
trả về `401` với error shape chung. Client phải dùng cặp token do refresh trả về.

### Data shape của auth response

Code block sau chỉ hiển thị object `data` bên trong auth response; không phải wire response đầy đủ:

```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token",
  "expiresIn": 900,
  "user": {
    "id": "user-uuid",
    "email": "customer@example.com",
    "name": "Customer",
    "role": "CUSTOMER"
  }
}
```

```http
POST /api/auth/refresh
Content-Type: application/json
```

```json
{
  "refreshToken": "jwt-refresh-token"
}
```

## 4. Catalog

### Liệt kê products

```http
GET /api/products?q=headphone&categoryId=category-uuid&minPrice=10&maxPrice=500&minRating=4&sort=relevance&page=0&size=20
```

Mọi query parameters đều optional:

| Parameter | Meaning |
|---|---|
| `q` | Text search trên product name và description |
| `categoryId` | Category filter |
| `minPrice` / `maxPrice` | Khoảng price bao gồm cận |
| `minRating` | Minimum rating từ `0` tới `5` |
| `sort` | `relevance`, `price_asc`, `price_desc` hoặc `rating_desc` |
| `page` | Zero-based page index; mặc định `0` |
| `size` | Page size; mặc định `20` |

Grid/list view là lựa chọn trình bày của frontend và không được gửi lên backend.

Sort mặc định là `relevance` khi có `q` và `rating_desc` trong các trường hợp còn lại. `relevance` là
backend search score, tiếp theo là `ratingAverage DESC`, `reviewCount DESC` và `id ASC` làm deterministic
tie-breakers. Các sorts khác dùng `id ASC` làm tie-breaker cuối cùng. Sort `popular` được hoãn lại cho đến khi
separate popularity metric được định nghĩa.

Wire response đầy đủ là `ApiResponse<PageResponse<ProductSummaryDto>>`:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": "product-uuid",
        "name": "Headphone",
        "description": "Wireless headphone",
        "imageUrl": "https://example.com/headphone.jpg",
        "price": "59.00",
        "stock": 12,
        "ratingAverage": "4.50",
        "reviewCount": 99,
        "category": {
          "id": "category-uuid",
          "name": "Electronics"
        },
        "sellerId": "seller-uuid"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

### Categories

```http
GET /api/categories
```

Wire response đầy đủ là `ApiResponse<List<CategorySummaryDto>>`:

```json
{
  "success": true,
  "data": [
    {
      "id": "category-uuid",
      "name": "Electronics"
    }
  ],
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

### Chi tiết product

```http
GET /api/products/{productId}
```

Response dùng cùng product fields như list response. MVP đầu tiên hỗ trợ một
`imageUrl` và không chọn color/size variant.

Wire response đầy đủ là `ApiResponse<ProductDto>`:

```json
{
  "success": true,
  "data": {
    "id": "product-uuid",
    "name": "Headphone",
    "description": "Wireless headphone",
    "imageUrl": "https://example.com/headphone.jpg",
    "price": "59.00",
    "stock": 12,
    "ratingAverage": "4.50",
    "reviewCount": 99,
    "category": {
      "id": "category-uuid",
      "name": "Electronics"
    },
    "sellerId": "seller-uuid"
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

## 5. Cart

Mọi cart endpoints yêu cầu token `CUSTOMER`. Server suy ra cart owner từ token.

```http
GET /api/cart
```

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "cart-item-uuid",
        "productId": "product-uuid",
        "name": "Headphone",
        "imageUrl": "https://example.com/headphone.jpg",
        "unitPrice": "59.00",
        "quantity": 2,
        "lineTotal": "118.00",
        "availableStock": 12,
        "sellerId": "seller-uuid"
      }
    ],
    "subtotal": "118.00"
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Wire response đầy đủ là `ApiResponse<CartDto>`.

```http
POST /api/cart/items
Content-Type: application/json
```

```json
{
  "productId": "product-uuid",
  "quantity": 1
}
```

```http
PATCH /api/cart/items/{cartItemId}
Content-Type: application/json
```

```json
{
  "quantity": 2
}
```

```http
DELETE /api/cart/items/{cartItemId}
```

Backend validate product activity và stock. Frontend không được coi displayed price là
authoritative checkout price. Nếu product đã có trong cart của customer, `POST` tăng
existing quantity thay vì tạo duplicate row. Combined quantity được validated với current stock.
Các mutations `POST`, `PATCH` và `DELETE` thành công trả về `ApiResponse<CartDto>` với status `200`
trong MVP đầu tiên; `INSUFFICIENT_STOCK` hoặc `CART_ITEM_NOT_FOUND` được trả về trong common error
envelope khi mutation không thể áp dụng.

## 6. Checkout và orders

### Tạo order

Server đọc cart của authenticated customer, validate products và stock, snapshot product
name/price, giảm hoặc giữ stock, và xóa purchased cart items trong một transaction.

```http
POST /api/orders
Authorization: Bearer <access-token>
Content-Type: application/json
```

```json
{
  "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
  "paymentMethod": "COD"
}
```

`paymentMethod` là `COD` cho MVP đầu tiên. `VNPAY` được dành cho payment slice tiếp theo.

Wire response đầy đủ là `ApiResponse<OrderDto>`:

```json
{
  "success": true,
  "data": {
    "id": "order-uuid",
    "status": "PENDING",
    "paymentStatus": "PENDING",
    "paymentMethod": "COD",
    "createdAt": "2026-10-12T10:30:00Z",
    "shippingAddress": "123 Nguyen Trai, District 1, Ho Chi Minh City",
    "shippingFee": "0.00",
    "discountAmount": "0.00",
    "totalAmount": "118.00",
    "items": [
      {
        "id": "order-item-uuid",
        "productName": "Headphone",
        "unitPrice": "59.00",
        "quantity": 2,
        "status": "PENDING",
        "sellerId": "seller-uuid"
      }
    ]
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

MVP money tính bằng `VND`, thể hiện dưới dạng decimal strings. Shipping cố định ở `"0.00"` trong slice đầu tiên;
order total do backend tính bằng item subtotal trừ discount cộng shipping.
Future prepayment deadline được suy ra từ `createdAt + 24 hours`; nó không phải domain field mới trong
class model hiện tại. Một prepayment order không completed trước deadline đó bị cancelled và
checkout stock của nó được restored đúng một lần. COD không có prepayment deadline.

Action `buy now` ở product-detail không có checkout contract riêng trong MVP đầu tiên. Nó thêm
selected product và quantity vào normal cart của authenticated customer, rồi điều hướng tới
standard checkout flow. Do đó order endpoint tiêu thụ toàn bộ current cart. Một
single-item checkout riêng có thể bổ sung sau nếu UX yêu cầu.

### Orders của customer

```http
GET /api/orders?page=0&size=20
GET /api/orders/{orderId}
```

Backend chỉ trả về orders thuộc sở hữu của authenticated customer. Shipment outcomes vẫn
seller-scoped bên trong order detail; một mixed seller result không được collapsed thành một false
order-level delivery status.

`GET /api/orders` trả về `ApiResponse<PageResponse<OrderSummaryDto>>`. Mỗi summary
chứa `id`, `status`, `paymentStatus`, `paymentMethod`, `createdAt` và `totalAmount`.
`GET /api/orders/{orderId}` trả về `ApiResponse<OrderDto>`; nó tái dùng các order-creation fields và thêm
`shipments: ShipmentDto[]`. Shipment DTO dưới đây do đó là một phần của order detail, không phải
standalone top-level response.

Order detail chứa nested `ShipmentDto` data shape sau. Đây không phải standalone top-level response và
không được trả về bên ngoài envelope `ApiResponse<OrderDto>`:

```json
{
  "id": "shipment-uuid",
  "orderId": "order-uuid",
  "sellerId": "seller-uuid",
  "deliveryMethod": "SELF_DELIVERY",
  "status": "SHIPPED",
  "trackingCode": "TRACK-001",
  "shippingFee": "0.00",
  "shippedAt": "2026-10-13T09:00:00Z",
  "deliveredAt": null,
  "items": [
    {
      "orderItemId": "order-item-uuid",
      "quantity": 2
    }
  ]
}
```

MVP đầu tiên ánh xạ whole order-item lines only. Response `quantity` là quantity từ
mapped `OrderItem`, không phải partial mapping quantity. Một order item thuộc về tối đa một shipment, và
`shipment.orderId == orderItem.orderId`; partial-line quantities, split shipments và replacement
shipments được hoãn lại. Một shipment phải chứa ít nhất một item, và mọi mapped items phải có cùng
seller với `shipment.sellerId`.

## 7. Reviews

Reviews publicly readable. Chỉ authenticated customer đã purchased và received product
mới được tạo một review cho product đó.

```http
GET /api/products/{productId}/reviews?page=0&size=20
```

```http
POST /api/products/{productId}/reviews
Authorization: Bearer <access-token>
Content-Type: application/json
```

```json
{
  "rating": 5,
  "comment": "Good product and fast delivery"
}
```

Review tags và image attachments không thuộc hợp đồng MVP đầu tiên.

Tạo review thành công trả về `201` với `ApiResponse<ReviewDto>`:

```json
{
  "success": true,
  "data": {
    "id": "review-uuid",
    "productId": "product-uuid",
    "userId": "customer-uuid",
    "rating": 5,
    "comment": "Good product and fast delivery",
    "createdAt": "2026-10-14T10:30:00Z"
  },
  "error": null,
  "timestamp": "2026-10-14T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Review listing trả về `ApiResponse<PageResponse<ReviewDto>>`; `data` chứa `content`, `page`, `size`,
`totalElements` và
`totalPages`. Một ineligible customer nhận
`REVIEW_NOT_ELIGIBLE`; một second review cho cùng customer/product nhận `REVIEW_ALREADY_EXISTS`.

## 8. AI product search

Mobile client gửi natural language tới backend. Backend ánh xạ request thành constrained
catalog query và sinh answer grounded trong catalog result.

```http
POST /api/ai/ask
Content-Type: application/json
```

```json
{
  "message": "Find me a black headphone under 2,500,000 VND with good reviews",
  "sessionId": null
}
```

```json
{
  "success": true,
  "data": {
    "answer": "I found 1 headphone under 2,500,000 VND with a rating of at least 4 stars.",
    "query": {
      "text": "black headphone",
      "maxPrice": "2500000.00",
      "minRating": "4"
    },
    "products": [
      {
        "id": "product-uuid",
        "name": "Black Wireless Headphone",
        "imageUrl": "https://example.com/black-headphone.jpg",
        "price": "89.00",
        "ratingAverage": "4.60",
        "reviewCount": 42,
        "sellerId": "seller-uuid"
      }
    ],
    "sessionId": null
  },
  "error": null,
  "timestamp": "2026-10-12T10:30:00Z",
  "traceId": "request-trace-id"
}
```

Wire response đầy đủ là `ApiResponse<AiProductSearchResponse>`.

Answer phải mô tả returned list; backend vẫn là source of truth cho actual
count và product data.

Guest requests có thể stateless. AI không được nhận provider API keys, arbitrary user IDs hoặc
write-capability tools. Vì MVP currency là `VND`, foreign-currency input phải được clarified hoặc
rejected; backend không được silently convert nó khi chưa có explicit exchange-rate policy.

## 9. State rules backend bắt buộc

```text
Prepayment order: PENDING -> PAID -> DELIVERED
COD order:        PENDING -> DELIVERED
Prepayment order: PENDING -> CANCELLED after the prepayment deadline

Shipment: PENDING -> PREPARING -> SHIPPED -> DELIVERED
Shipment: SHIPPED -> FAILED -> RETURNED
Shipment: PENDING/PREPARING -> CANCELLED (before payment completion)
```

- `FAILED` không khôi phục stock.
- Chỉ transition `RETURNED` thành công mới khôi phục shipped quantity cho seller đó.
- Một multi-seller order có thể có independent shipment outcomes.
- COD được tạo ở `PENDING` và chỉ marked `COMPLETED` sau khi mọi required shipments đều
  `DELIVERED`; transition `DELIVERED` required cuối cùng là collection trigger do
  shipment/payment services xử lý trong một transaction. Nếu bất kỳ seller shipment nào trở thành `FAILED` hoặc `RETURNED`,
  không COD collection nào được ghi nhận và MVP không thực hiện partial settlement hoặc refund.
- Direct order-item cancellation chỉ được phép trước shipment assignment. Với assigned
  `PENDING`/`PREPARING` shipment, seller phải cancel shipment thay thế; atomic operation đó
  marks các mapped whole order-item lines thành `CANCELLED` và releases unshipped stock của chúng đúng một lần.
  Cancellation sau `SHIPPED` hoặc bất kỳ later shipment state nào bị rejected. Shipment cancellation sau khi
  prepayment đã completed cũng bị rejected cho đến khi có refund policy.
- `OrderStatus.DELIVERED` chỉ xét non-cancelled order items và chỉ đạt được khi mọi required
  shipments của chúng đã delivered. Do đó COD order có thể đi từ `PENDING` trực tiếp tới
  `DELIVERED` trong khi payment của nó đi từ `PENDING` tới `COMPLETED`.
- Khi seller shipments có mixed terminal outcomes, order vẫn là container cho các seller-
  scoped shipment results; clients phải hiển thị `DELIVERED`, `FAILED` hoặc `RETURNED` theo từng shipment và
  không được invent order-level success/failure enum mới.
- Với valid cancellation trước payment completion, order service tính lại `totalAmount` và
  `amount` của mọi still-`PENDING` payment atomically từ các remaining non-cancelled order-item
  snapshots cộng fixed shipping fee. Nó không bao giờ reread current product price; cancelled line
  snapshots giữ nguyên cho history.
- Sau cancellation, aggregate state được reevaluated với non-empty guard: nếu mọi order items đều
  `CANCELLED`, order trở thành `CANCELLED` và không COD collection nào được ghi nhận; nếu non-cancelled items
  còn lại và mọi required shipments của chúng đã `DELIVERED`, order/payment có thể complete trong
  cùng cancellation transaction; ngược lại order vẫn pending.
- Payment callbacks, khi VNPAY được thêm vào, phải idempotent.

## 10. Hợp đồng hoãn lại

Các interfaces sau cố ý chưa frozen cho MVP đầu tiên:

- VNPAY request/callback payloads;
- voucher validation và application;
- quản lý seller product/order/shipment;
- Admin operations;
- human chat WebSocket messages;
- product variants và variant stock;
- review tags và image uploads;
- persisted AI sessions và customer order/shipment AI tools.
