# Kiểm thử

## 1. Mục đích và Phạm vi

Tài liệu này mô tả các automated test hiện có trong repository
`commerce-platform-frontend-mobile` (client React Native / Expo) và kết quả đã được kiểm chứng cho
frontend revision được ghi ở mục 6.

Phạm vi automated frontend hiện đã kiểm chứng bao gồm:

- utility và business logic (định dạng VND, mapping DTO backend → product UI, đặt tên query key);
- các React Native component dùng lại (`Button`, `PriceDisplay`, `ProductImage`);
- hành vi Cart Context / hook (cart chạy tại runtime cục bộ);
- hành vi render và điều hướng giới hạn của `CartScreen` (chỉ trạng thái rỗng).

Cột mốc này **không** bao gồm:

- backend integration test;
- system test;
- acceptance test;
- real server Cart test (Cart API backend chưa tồn tại).

Cart trên frontend được thiết kế là state runtime cục bộ do `CartContext` quản lý; đây không phải
cart trên server. Không nội dung nào trong tài liệu này được hiểu là bằng chứng của việc kiểm chứng
backend, integration, system hay acceptance.

## 2. Chiến lược Kiểm thử

Các test đã triển khai được phân thành bốn nhóm. Việc phân loại phản ánh đúng điều mà mỗi test thực
sự làm, không phải thư mục chứa nó.

| Nhóm | Số lượng | Mô tả |
|---|---|---|
| Unit Test | 18 | Hàm thuần và key factory, không render và không I/O. |
| Component Test | 13 | Một React Native component được render, truy vấn và điều khiển bằng event. |
| Hook/Context Test | 12 | Kiểm tra contract state/action của React qua `renderHook` với provider thật. |
| Screen/Rendering Test | 2 | Một screen được render trong provider với prop navigation được stub. |
| **Tổng** | **45** | |

Vì sao `CartContext` và `CartScreen` **không** phải integration test:

- Test `CartContext` kiểm tra state React trong bộ nhớ thông qua hook công khai `useCart`. Không có
  network call, không có backend thật, không có database.
- Test `CartScreen` render một screen trong `CartProvider` thật nhưng stub prop navigation. Không có
  navigation stack, không có route resolution và không có request tới server.

Vì không test nào vượt qua ranh giới module thật để tới một backend, navigator hay database đang
chạy, không test nào trong số này đủ điều kiện là integration, system hay acceptance test.

## 3. Môi trường và Công cụ Kiểm thử

Stack kiểm thử được ghi trong `package.json` và `package-lock.json` của frontend. Không có package
test nào khác ngoài danh sách dưới đây được sử dụng.

### Baseline runtime và ngôn ngữ

| Công cụ | Phiên bản |
|---|---|
| Expo | `~57.0.30` |
| React | `19.2.3` |
| React Native | `0.86.3` |
| TypeScript | `~6.0.3` |

### Dependency kiểm thử (`devDependencies`)

| Package | Phiên bản | Vai trò |
|---|---|---|
| `jest` | `~29.7.0` | Test runner |
| `jest-expo` | `~57.0.6` | Jest preset do Expo quản lý |
| `@types/jest` | `29.5.14` | Typings cho global của Jest |
| `@testing-library/react-native` | `12.8.1` | Render và query cho component/hook |
| `react-test-renderer` | `19.2.3` | Renderer được ghim khớp với React |

### Cấu hình

| File | Mục đích |
|---|---|
| `jest.config.js` | Đặt `preset: 'jest-expo'`, `setupFilesAfterEnv: ['<rootDir>/jest.setup.js']` và `testMatch: ['**/__tests__/**/*.test.(ts|tsx|js)']`. |
| `jest.setup.js` | Đăng ký matcher của React Native Testing Library qua `@testing-library/react-native/extend-expect`. |
| `tsconfig.json` | Thêm `"jest"` vào `types` để test file type-check được bằng `npm run typecheck`. |

Test được đặt cùng chỗ dưới `src/**/__tests__/`. Không có thư mục mocks, fixtures hay helpers riêng;
mỗi test tự định nghĩa fixture nhỏ inline. `coverage/` được git-ignore.

## 4. Cách Chạy Test

Chạy tất cả lệnh trong thư mục `commerce-platform-frontend-mobile`.

| Lệnh | Mục đích |
|---|---|
| `npm test` | Chạy toàn bộ Jest suite một lần. |
| `npm run test:coverage` | Chạy suite và in báo cáo coverage. |
| `npm run typecheck` | Chạy `tsc --noEmit` để kiểm tra TypeScript. |
| `npm run test:watch` | Chế độ watch tương tác khi phát triển cục bộ. **Không** phải bằng chứng CI. |

`test:watch` chỉ dành cho tiện lợi khi lập trình; đây là lệnh tương tác và không được dùng làm kết
quả kiểm chứng.

Các bước tái lập:

1. Cài dependency khớp `package-lock.json` (`npm install` hoặc `npm ci`).
2. Chạy `npm run typecheck` và xác nhận không có lỗi.
3. Chạy `npm test` và xác nhận tất cả suite pass.
4. Chạy `npm run test:coverage` để tái lập các số liệu coverage ở mục 7.

Các bước này chỉ chạy script hiện có; không cần thay đổi code frontend.

## 5. Đặc tả Test Case

Mỗi dòng dưới đây tương ứng đúng một case `it()` trong source test của frontend. Cột **Source** ghi
đường dẫn test file tương đối tính từ thư mục gốc repository frontend. Test case không có tham chiếu
requirement hợp lệ được đánh dấu `No verified requirement reference`; requirement ID chỉ lấy từ SRS và
không bao giờ được tự tạo.

### 5.1 Utility — `formatVND`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-UT-FMT-01 | Unit | `formatVND` | Nhóm hàng nghìn bằng dấu chấm và thêm ký hiệu tiền tệ | `formatVND(29990000)`, `formatVND(29900000)` | `"29.990.000 ₫"`, `"29.900.000 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-02 | Unit | `formatVND` | Định dạng số dưới 1000 không có dấu phân cách | `formatVND(500)` | `"500 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-03 | Unit | `formatVND` | Định dạng đúng một nghìn | `formatVND(1000)` | `"1.000 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-04 | Unit | `formatVND` | Xử lý số 0 | `formatVND(0)` | `"0 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-05 | Unit | `formatVND` | Làm tròn số thập phân về số nguyên gần nhất | `formatVND(1500.4)`, `formatVND(1500.6)` | `"1.500 ₫"`, `"1.501 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-06 | Unit | `formatVND` | Đặt dấu âm trước các chữ số đã nhóm cho số âm | `formatVND(-1250000)` | `"-1.250.000 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |

### 5.2 Utility — `productMapper`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-UT-MAP-01 | Unit | `mapProductSummaryToProduct` | Map mọi field scalar từ DTO | Một `ProductSummaryDto` cố định | `id`, `name`, `description`, `category`, `sellerId`, `reviewCount`, `stock` được sao chép qua | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-02 | Unit | `mapProductSummaryToProduct` | Parse chuỗi thập phân thành số | DTO với `price: "29990000.00"`, `ratingAverage: "4.70"` | `price === 29990000`, `rating ≈ 4.7` | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-03 | Unit | `mapProductSummaryToProduct` | Dùng nguồn uri từ xa khi có `imageUrl` | DTO với `imageUrl: "https://cdn.example.com/iphone.jpg"` | `image` bằng `{ uri: "<cùng url>" }` | FR-CAT-01 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-04 | Unit | `mapProductSummaryToProduct` | Fallback về placeholder đi kèm khi `imageUrl` là `null` | DTO với `imageUrl: null` | `image` được định nghĩa và không phải nguồn `{ uri }` từ xa | FR-CAT-01 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-05 | Unit | `mapProductSummaryToProduct` | Fallback về placeholder đi kèm khi `imageUrl` rỗng | DTO với `imageUrl: "   "` | `image` được định nghĩa và không phải nguồn `{ uri }` từ xa | FR-CAT-01 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-06 | Unit | `mapProductSummaryToProduct` | Trả về `0` cho giá không phải số thay vì `NaN` | DTO với `price: "not-a-number"` | `price === 0` | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-07 | Unit | `mapProductSummaryToProduct` | Luôn để `originalPrice` là undefined (backend không cung cấp) | Một `ProductSummaryDto` cố định | `originalPrice === undefined` | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-08 | Unit | `mapProductDetailToProduct` | Map detail DTO giống như summary DTO | Một `ProductDetailDto` với ảnh từ xa và giá/rating khác | id/image/price/rating/stock được map đúng như mong đợi | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |

### 5.3 Utility — `queryKeys`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-UT-QK-01 | Unit | `queryKeys` | Mọi query list/detail nằm dưới prefix resource của nó | `user.me()`, `products.detail('p-1')`, `cart.current()`, `orders.detail('o-1')` | Phần tử key đầu tiên bằng tên resource | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |
| TC-UT-QK-02 | Unit | `queryKeys` | Dùng prefix chỉ-resource ổn định cho invalidation | `.all` cho products, categories, sellers, reviews, shipments | Mỗi `.all` bằng mảng một phần tử tên resource | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |
| TC-UT-QK-03 | Unit | `queryKeys` | Nhúng filter vào product list key để các tổ hợp khác nhau | `products.list({ q, page })` với page 0 vs 1 | Key chứa object filter và khác nhau giữa các page | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |
| TC-UT-QK-04 | Unit | `queryKeys` | Chứa identifier trong detail key | Các factory detail/shop/byProduct/byOrder | Mỗi key chứa identifier của nó | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |

### 5.4 Hook/Context — `CartContext`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CTX-CART-01 | Hook/Context | `useCart` / `CartProvider` | Bắt đầu rỗng | Render hook trong `CartProvider` mới | `items === []`, `totalItems === 0`, `subtotal === 0` | FR-CART-01 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-02 | Hook/Context | `addItem` | Thêm sản phẩm với quantity mặc định là 1 | `addItem(product)` | Một dòng, quantity 1, `totalItems === 1`, `subtotal === 1000000` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-03 | Hook/Context | `addItem` | Thêm sản phẩm với quantity chỉ định | `addItem(product, 3)` | Quantity 3, `subtotal === 3000000` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-04 | Hook/Context | `addItem` | Gộp sản phẩm lặp lại thay vì tạo dòng mới | `addItem(product, 1)` rồi `addItem(product, 2)` | Một dòng, quantity 3 | FR-CART-03 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-05 | Hook/Context | `addItem` | Giữ các sản phẩm khác nhau ở dòng riêng | `addItem(p-1, 1000)`, `addItem(p-2, 2000)` | Hai dòng, `subtotal === 3000` | FR-CART-03 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-06 | Hook/Context | `addItem` | Làm tròn xuống quantity thập phân và không bao giờ dưới 1 | `addItem(p-1, 2.9)`, `addItem(p-2, 0)` | Quantity 2 và 1 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-07 | Hook/Context | `increaseQuantity` | Tăng quantity của một dòng đã biết | `addItem(product)`, `increaseQuantity('p-1')` | Quantity 2, `totalItems === 2` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-08 | Hook/Context | `decreaseQuantity` | Giảm quantity xuống sàn là 1 | `addItem(product, 2)`, `decreaseQuantity` hai lần | Quantity giữ nguyên 1 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-09 | Hook/Context | `increaseQuantity` / `decreaseQuantity` | Bỏ qua increase/decrease với product id không tồn tại | `increaseQuantity('missing')`, `decreaseQuantity('missing')` | Cart không đổi, quantity 1 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-10 | Hook/Context | `removeItem` | Xóa một dòng bất kể quantity | `addItem(product, 5)`, `removeItem('p-1')` | `items === []`, `totalItems === 0` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-11 | Hook/Context | `clearCart` | Xóa toàn bộ cart | Thêm hai sản phẩm, `clearCart()` | `items === []`, tổng bằng 0 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-12 | Hook/Context | `useCart` | Throw khi dùng ngoài `CartProvider` | `renderHook(() => useCart())` không có provider | Throw `"useCart must be used within a CartProvider"` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |

### 5.5 Component — `Button`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CT-BTN-01 | Component | `Button` | Render tiêu đề của nó | `<Button title="Add to cart" />` | Text `"Add to cart"` xuất hiện | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-02 | Component | `Button` | Gọi `onPress` khi được nhấn | Nhấn button | `onPress` được gọi một lần | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-03 | Component | `Button` | Không gọi `onPress` khi bị disable | Nhấn button `disabled` | `onPress` không được gọi | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-04 | Component | `Button` | Không gọi `onPress` khi đang loading | Nhấn button `loading` | `onPress` không được gọi | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-05 | Component | `Button` | Expose accessibility state busy/disabled khi loading | Đọc `accessibilityState` của button loading | State khớp `{ busy: true, disabled: true }` | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-06 | Component | `Button` | Dùng `accessibilityLabel` ghi đè khi được cung cấp | `<Button accessibilityLabel="Increase quantity" />` | Tìm được phần tử theo label đó | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |

### 5.6 Component — `PriceDisplay`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CT-PRC-01 | Component | `PriceDisplay` | Render giá hiện tại được định dạng theo VND | `<PriceDisplay price={29990000} />` | `"29.990.000 ₫"` được render | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |
| TC-CT-PRC-02 | Component | `PriceDisplay` | Không render giá gốc khi không được cung cấp | `<PriceDisplay price={29990000} />` | Không có node `"35.000.000 ₫"` | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |
| TC-CT-PRC-03 | Component | `PriceDisplay` | Render cả hai giá khi giá gốc cao hơn | `<PriceDisplay price={29990000} originalPrice={35000000} />` | Cả hai giá đã định dạng được render | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |
| TC-CT-PRC-04 | Component | `PriceDisplay` | Ẩn giá gốc khi nó không cao hơn giá hiện tại | `<PriceDisplay price={29990000} originalPrice={29990000} />` | Chỉ giá hiện tại được render | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |

### 5.7 Component — `ProductImage`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CT-IMG-01 | Component | `ProductImage` | Render `Image` cho nguồn hợp lệ | `<ProductImage source={{ uri }} />` | `Image` được render nhận đúng source đó | FR-CAT-01 | `src/components/__tests__/ProductImage.test.tsx` |
| TC-CT-IMG-02 | Component | `ProductImage` | Hiện placeholder sau khi ảnh tải thất bại | Fire `error` trên `Image` | Placeholder `🖼️` được render | FR-CAT-01 | `src/components/__tests__/ProductImage.test.tsx` |
| TC-CT-IMG-03 | Component | `ProductImage` | Khôi phục `Image` khi source thay đổi sau thất bại | Fire `error`, rồi rerender với source mới | Placeholder biến mất; `Image` có source mới | FR-CAT-01 | `src/components/__tests__/ProductImage.test.tsx` |

### 5.8 Screen/Rendering — `CartScreen`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-SC-CART-01 | Screen/Rendering | `CartScreen` | Render trạng thái rỗng khi cart không có item | Render trong `CartProvider` rỗng | `"Your cart is empty"` và `"Continue Shopping"` được hiển thị | FR-CART-01 | `src/screens/__tests__/CartScreen.test.tsx` |
| TC-SC-CART-02 | Screen/Rendering | `CartScreen` | Điều hướng về Home khi nhấn Continue Shopping | Nhấn control `Continue shopping` | `navigation.navigate('Home')` được gọi | FR-CART-01 | `src/screens/__tests__/CartScreen.test.tsx` |

## 6. Kết quả Kiểm thử Đã kiểm chứng

Kết quả dưới đây là kết quả đã kiểm chứng cho frontend revision được ghi lại. Đây không phải bảo
đảm toàn dự án vĩnh viễn và sẽ thay đổi khi bộ test phát triển.

| Trường | Giá trị |
|---|---|
| Frontend commit | `36e02a6` |
| Merge commit | `866897b` |
| Test suite | 8 pass / 8 tổng |
| Test | 45 pass / 45 tổng |
| Failed | 0 |
| Skipped | 0 |
| Snapshots | 0 |

Lệnh thực thi: `npm test`.

## 7. Coverage

Coverage là bằng chứng tại một thời điểm được tạo bởi `npm run test:coverage`, gắn với frontend
revision ở mục 6. Con số statements tổng `82.82%` bên dưới **không** phải bảo đảm chất lượng toàn dự
án vĩnh viễn.

### Tổng quan

| Chỉ số | Phần trăm |
|---|---|
| Statements | 82.82% |
| Branches | 65.82% |
| Functions | 73.58% |
| Lines | 81.72% |

### Các module được đo

| File | Statements % | Branches % | Functions % | Lines % |
|---|---|---|---|---|
| `utils/format.ts` | 100 | 100 | 100 | 100 |
| `utils/productMapper.ts` | 100 | 78.57 | 100 | 100 |
| `context/CartContext.tsx` | 100 | 90.9 | 100 | 100 |
| `hooks/useCart.ts` | 100 | 100 | 100 | 100 |
| `components/Button.tsx` | 100 | 85.71 | 100 | 100 |
| `components/PriceDisplay.tsx` | 100 | 100 | 100 | 100 |
| `components/ProductImage.tsx` | 100 | 100 | 100 | 100 |
| `api/queryKeys.ts` | 81.81 | 0 | 80 | 81.81 |
| `screens/CartScreen.tsx` | 47.05 | 16.66 | 27.27 | 43.75 |
| `components/CartItemRow.tsx` | 14.28 | 0 | 0 | 14.28 |

## 8. Truy vết Requirement

Chỉ các requirement ID tồn tại trong SRS (`02-requirements-specification-srs/`) mới được dùng. Khi
không có tham chiếu hợp lệ, test được đánh dấu `No verified requirement reference`.

| Requirement ID | Requirement | Frontend test |
|---|---|---|
| FR-CART-01 | Khách hàng phải xem được cart. | TC-SC-CART-01, TC-SC-CART-02, TC-CTX-CART-01 |
| FR-CART-02 | Khách hàng phải thêm, cập nhật và xóa được cart item. | TC-CTX-CART-02, TC-CTX-CART-03, TC-CTX-CART-06, TC-CTX-CART-07, TC-CTX-CART-08, TC-CTX-CART-09, TC-CTX-CART-10, TC-CTX-CART-11, TC-CTX-CART-12 |
| FR-CART-03 | Cart của khách hàng chỉ chứa tối đa một dòng cho một sản phẩm. | TC-CTX-CART-04, TC-CTX-CART-05 |
| FR-CAT-03 | Khách hàng phải xem được sản phẩm và category. | TC-UT-MAP-01, TC-UT-MAP-02, TC-UT-MAP-06, TC-UT-MAP-07, TC-UT-MAP-08, TC-CT-PRC-01, TC-CT-PRC-02, TC-CT-PRC-03, TC-CT-PRC-04 |
| FR-CAT-01 | Khách hàng phải duyệt được sản phẩm. | TC-UT-MAP-03, TC-UT-MAP-04, TC-UT-MAP-05, TC-CT-IMG-01, TC-CT-IMG-02, TC-CT-IMG-03 |

Ghi chú:

- Các test utility `formatVND`, các test `queryKeys` và các test `Button` chung **không** được map
  vào requirement nào; không có tham chiếu hợp lệ nên không tự tạo.
- FR-CART-04 và FR-CART-05 (validate stock phía server và tính lại giá phía server) **không** được
  phủ bởi automated test frontend; những rule này thuộc backend và Cart API backend chưa tồn tại.

### Không được phủ (nêu rõ)

| Requirement ID | Trạng thái |
|---|---|
| NFR-SEC-03 | Không được phủ bởi automated test frontend hiện tại. |

## 9. Hạn chế và Kiểm thử Tương lai

Các automated test frontend hiện tại **không** kiểm chứng:

- tích hợp backend thật;
- hành vi Cart phía server;
- checkout;
- orders;
- payment / VNPAY;
- shipment;
- reviews;
- voucher;
- chat;
- AI;
- notifications;
- luồng phê duyệt Seller Product Management.

Các feature này chưa được triển khai hoặc kiểm thử trong cột mốc frontend hiện tại và không được mô
tả là đã hoàn tất.

Các khoảng trống bổ sung đã biết trong phạm vi đã test:

- `CartScreen` chỉ được phủ ở trạng thái rỗng; danh sách item, control quantity, summary và
  placeholder checkout chưa được test.
- `CartItemRow` chưa được test trực tiếp.
- Branch coverage của `queryKeys` là `0%` trong lần chạy được ghi lại; các assertion kiểm tra hình
  dạng key, không phải mọi branch.

## 10. Bằng chứng

Thư mục `04-testing/evidence/` được dành cho bằng chứng thu thập thủ công. Chưa có ảnh chụp nào được
commit; các file liệt kê dưới đây hiện **chưa** tồn tại.

Bằng chứng cần thu thập sau này:

| Bằng chứng | Tên file gợi ý | Thu thập từ |
|---|---|---|
| Jest PASS summary | `evidence/jest-pass-<commit>.png` | Output của `npm test` |
| Coverage summary | `evidence/jest-coverage-<commit>.png` | `npm run test:coverage` |
| Typecheck pass | `evidence/typecheck-pass-<commit>.png` | `npm run typecheck` |
| Hành vi app tùy chọn | `evidence/app-cart-empty-<commit>.png` | App đang chạy, cart rỗng |

Mỗi mục thu thập cần được ghi nhãn kèm frontend commit và ngày tạo.
