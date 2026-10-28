# Testing

## 1. Purpose and Scope

This document describes the automated tests that exist today in the
`commerce-platform-frontend-mobile` repository (React Native / Expo client) and the verified
results captured for the frontend revision documented in section 6.

The current verified automated frontend scope covers:

- utility and business logic (VND formatting, backend DTO → UI product mapping, query-key naming);
- reusable React Native components (`Button`, `PriceDisplay`, `ProductImage`);
- Cart Context / hook behavior (the local runtime cart);
- limited `CartScreen` rendering and navigation behavior (empty state only).

This milestone explicitly does **not** contain:

- backend integration tests;
- system tests;
- acceptance tests;
- real server Cart tests in this frontend milestone. The backend Cart API now exists; see
  [backend verification](BACKEND_VERIFICATION.md) for the separate backend baseline.

The frontend cart is intentionally a local runtime state owned by `CartContext`; it is not a server
cart. Nothing in this document should be read as evidence of backend, integration, system, or
acceptance verification.

## 2. Test Strategy

The implemented tests are classified into four categories. The classification reflects what each
test actually does, not the folder it lives in.

| Category | Count | Description |
|---|---|---|
| Unit Test | 18 | Pure functions and key factories with no rendering or I/O. |
| Component Test | 13 | An isolated React Native component rendered, queried, and driven by events. |
| Hook/Context Test | 12 | React state/action contract via `renderHook` with a real provider. |
| Screen/Rendering Test | 2 | A screen rendered inside a provider with a stubbed navigation prop. |
| **Total** | **45** | |

Why `CartContext` and `CartScreen` are **not** integration tests:

- `CartContext` tests exercise in-memory React state through the public `useCart` hook. No network
  call, no real backend, and no database are involved.
- `CartScreen` tests render a screen inside a real `CartProvider` but stub the navigation prop. No
  navigation stack, no route resolution, and no server request are exercised.

Because no test crosses a real module boundary against a live backend, navigator, or database, none
of these tests qualify as integration, system, or acceptance tests.

## 3. Test Environment and Tools

The testing stack is recorded in the frontend `package.json` and `package-lock.json`. No test
package beyond the list below is used.

### Runtime and language baseline

| Tool | Version |
|---|---|
| Expo | `~57.0.30` |
| React | `19.2.3` |
| React Native | `0.86.3` |
| TypeScript | `~6.0.3` |

### Testing dependencies (`devDependencies`)

| Package | Version | Role |
|---|---|---|
| `jest` | `~29.7.0` | Test runner |
| `jest-expo` | `~57.0.6` | Expo-managed Jest preset |
| `@types/jest` | `29.5.14` | Jest global typings |
| `@testing-library/react-native` | `12.8.1` | Component/hook rendering and queries |
| `react-test-renderer` | `19.2.3` | Renderer pinned to match React |

### Configuration

| File | Purpose |
|---|---|
| `jest.config.js` | Sets `preset: 'jest-expo'`, `setupFilesAfterEnv: ['<rootDir>/jest.setup.js']`, and `testMatch: ['**/__tests__/**/*.test.(ts|tsx|js)']`. |
| `jest.setup.js` | Registers React Native Testing Library matchers via `@testing-library/react-native/extend-expect`. |
| `tsconfig.json` | Adds `"jest"` to `types` so test files type-check with `npm run typecheck`. |

Tests are co-located under `src/**/__tests__/`. No separate mocks, fixtures, or helpers directories
exist; individual tests define their own small inline fixtures. `coverage/` is git-ignored.

## 4. Test Execution

Run all commands from the `commerce-platform-frontend-mobile` directory.

| Command | Purpose |
|---|---|
| `npm test` | Runs the full Jest suite once. |
| `npm run test:coverage` | Runs the suite and prints a coverage report. |
| `npm run typecheck` | Runs `tsc --noEmit` to validate TypeScript. |
| `npm run test:watch` | Local interactive watch mode. **Not** CI evidence. |

`test:watch` is a developer convenience only; it is interactive and must not be cited as a
verification result.

Reproducibility:

1. Install dependencies matching `package-lock.json` (`npm install` or `npm ci`).
2. Run `npm run typecheck` and confirm it reports no errors.
3. Run `npm test` and confirm all suites pass.
4. Run `npm run test:coverage` to reproduce the coverage figures in section 7.

These steps only run existing scripts; they require no frontend code changes.

## 5. Test Case Specification

Every row below corresponds to exactly one `it()` case in the frontend test source. The **Source**
column gives the test file path relative to the frontend repository root. Test cases without a
legitimate requirement reference are marked `No verified requirement reference`; requirement IDs are
taken only from the SRS and are never invented.

### 5.1 Utility — `formatVND`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-UT-FMT-01 | Unit | `formatVND` | Groups thousands with dots and appends the currency symbol | `formatVND(29990000)`, `formatVND(29900000)` | `"29.990.000 ₫"`, `"29.900.000 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-02 | Unit | `formatVND` | Formats amounts below 1000 without separators | `formatVND(500)` | `"500 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-03 | Unit | `formatVND` | Formats exactly one thousand | `formatVND(1000)` | `"1.000 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-04 | Unit | `formatVND` | Handles zero | `formatVND(0)` | `"0 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-05 | Unit | `formatVND` | Rounds fractional amounts to the nearest integer | `formatVND(1500.4)`, `formatVND(1500.6)` | `"1.500 ₫"`, `"1.501 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |
| TC-UT-FMT-06 | Unit | `formatVND` | Places the sign before the grouped digits for negative amounts | `formatVND(-1250000)` | `"-1.250.000 ₫"` | No verified requirement reference | `src/utils/__tests__/format.test.ts` |

### 5.2 Utility — `productMapper`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-UT-MAP-01 | Unit | `mapProductSummaryToProduct` | Maps every scalar field from the DTO | A fixed `ProductSummaryDto` | `id`, `name`, `description`, `category`, `sellerId`, `reviewCount`, `stock` copied through | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-02 | Unit | `mapProductSummaryToProduct` | Parses decimal strings into numbers | DTO with `price: "29990000.00"`, `ratingAverage: "4.70"` | `price === 29990000`, `rating ≈ 4.7` | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-03 | Unit | `mapProductSummaryToProduct` | Uses a remote uri source when `imageUrl` is present | DTO with `imageUrl: "https://cdn.example.com/iphone.jpg"` | `image` equals `{ uri: "<same url>" }` | FR-CAT-01 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-04 | Unit | `mapProductSummaryToProduct` | Falls back to the bundled placeholder when `imageUrl` is `null` | DTO with `imageUrl: null` | `image` is defined and is not a remote `{ uri }` source | FR-CAT-01 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-05 | Unit | `mapProductSummaryToProduct` | Falls back to the bundled placeholder when `imageUrl` is blank | DTO with `imageUrl: "   "` | `image` is defined and is not a remote `{ uri }` source | FR-CAT-01 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-06 | Unit | `mapProductSummaryToProduct` | Returns `0` for a non-numeric price instead of `NaN` | DTO with `price: "not-a-number"` | `price === 0` | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-07 | Unit | `mapProductSummaryToProduct` | Always leaves `originalPrice` undefined (not supplied by the backend) | A fixed `ProductSummaryDto` | `originalPrice === undefined` | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |
| TC-UT-MAP-08 | Unit | `mapProductDetailToProduct` | Maps the detail DTO the same way as the summary DTO | A `ProductDetailDto` with a remote image and different price/rating | Mapped id/image/price/rating/stock as expected | FR-CAT-03 | `src/utils/__tests__/productMapper.test.ts` |

### 5.3 Utility — `queryKeys`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-UT-QK-01 | Unit | `queryKeys` | Keys every list/detail query under its resource prefix | `user.me()`, `products.detail('p-1')`, `cart.current()`, `orders.detail('o-1')` | First key element equals the resource name | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |
| TC-UT-QK-02 | Unit | `queryKeys` | Uses stable resource-only prefixes for invalidation | `.all` for products, categories, sellers, reviews, shipments | Each `.all` equals a single-element resource array | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |
| TC-UT-QK-03 | Unit | `queryKeys` | Embeds filters in the product list key so combinations differ | `products.list({ q, page })` with page 0 vs 1 | Keys include the filter object and differ across pages | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |
| TC-UT-QK-04 | Unit | `queryKeys` | Includes identifiers in detail keys | detail/shop/byProduct/byOrder factories | Each key contains its identifier | No verified requirement reference | `src/api/__tests__/queryKeys.test.ts` |

### 5.4 Hook/Context — `CartContext`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CTX-CART-01 | Hook/Context | `useCart` / `CartProvider` | Starts empty | Render hook inside a fresh `CartProvider` | `items === []`, `totalItems === 0`, `subtotal === 0` | FR-CART-01 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-02 | Hook/Context | `addItem` | Adds a product with the default quantity of 1 | `addItem(product)` | One line, quantity 1, `totalItems === 1`, `subtotal === 1000000` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-03 | Hook/Context | `addItem` | Adds a product with an explicit quantity | `addItem(product, 3)` | Quantity 3, `subtotal === 3000000` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-04 | Hook/Context | `addItem` | Merges a repeated product instead of creating a new line | `addItem(product, 1)` then `addItem(product, 2)` | One line, quantity 3 | FR-CART-03 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-05 | Hook/Context | `addItem` | Keeps distinct products on separate lines | `addItem(p-1, 1000)`, `addItem(p-2, 2000)` | Two lines, `subtotal === 3000` | FR-CART-03 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-06 | Hook/Context | `addItem` | Floors fractional quantities and never adds below 1 | `addItem(p-1, 2.9)`, `addItem(p-2, 0)` | Quantities 2 and 1 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-07 | Hook/Context | `increaseQuantity` | Increases the quantity of a known line | `addItem(product)`, `increaseQuantity('p-1')` | Quantity 2, `totalItems === 2` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-08 | Hook/Context | `decreaseQuantity` | Decreases the quantity down to a floor of 1 | `addItem(product, 2)`, `decreaseQuantity` twice | Quantity stays 1 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-09 | Hook/Context | `increaseQuantity` / `decreaseQuantity` | Ignores increase/decrease for unknown product ids | `increaseQuantity('missing')`, `decreaseQuantity('missing')` | Cart unchanged, quantity 1 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-10 | Hook/Context | `removeItem` | Removes a line regardless of its quantity | `addItem(product, 5)`, `removeItem('p-1')` | `items === []`, `totalItems === 0` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-11 | Hook/Context | `clearCart` | Clears the whole cart | Add two products, `clearCart()` | `items === []`, totals 0 | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |
| TC-CTX-CART-12 | Hook/Context | `useCart` | Throws when used outside a `CartProvider` | `renderHook(() => useCart())` without a provider | Throws `"useCart must be used within a CartProvider"` | FR-CART-02 | `src/context/__tests__/CartContext.test.tsx` |

### 5.5 Component — `Button`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CT-BTN-01 | Component | `Button` | Renders its title | `<Button title="Add to cart" />` | Text `"Add to cart"` is present | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-02 | Component | `Button` | Calls `onPress` when pressed | Press the button | `onPress` called once | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-03 | Component | `Button` | Does not call `onPress` when disabled | Press a `disabled` button | `onPress` not called | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-04 | Component | `Button` | Does not call `onPress` while loading | Press a `loading` button | `onPress` not called | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-05 | Component | `Button` | Exposes a busy/disabled accessibility state when loading | Read `accessibilityState` of the loading button | State matches `{ busy: true, disabled: true }` | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |
| TC-CT-BTN-06 | Component | `Button` | Uses the `accessibilityLabel` override when provided | `<Button accessibilityLabel="Increase quantity" />` | Element found by that label | No verified requirement reference | `src/components/__tests__/Button.test.tsx` |

### 5.6 Component — `PriceDisplay`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CT-PRC-01 | Component | `PriceDisplay` | Renders the current price formatted in VND | `<PriceDisplay price={29990000} />` | `"29.990.000 ₫"` rendered | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |
| TC-CT-PRC-02 | Component | `PriceDisplay` | Does not render an original price when none is given | `<PriceDisplay price={29990000} />` | No `"35.000.000 ₫"` node | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |
| TC-CT-PRC-03 | Component | `PriceDisplay` | Renders both prices when the original price is higher | `<PriceDisplay price={29990000} originalPrice={35000000} />` | Both formatted prices rendered | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |
| TC-CT-PRC-04 | Component | `PriceDisplay` | Hides the original price when it is not higher than the current price | `<PriceDisplay price={29990000} originalPrice={29990000} />` | Only the current price is rendered | FR-CAT-03 | `src/components/__tests__/PriceDisplay.test.tsx` |

### 5.7 Component — `ProductImage`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-CT-IMG-01 | Component | `ProductImage` | Renders an `Image` for a valid source | `<ProductImage source={{ uri }} />` | Rendered `Image` receives that source | FR-CAT-01 | `src/components/__tests__/ProductImage.test.tsx` |
| TC-CT-IMG-02 | Component | `ProductImage` | Shows the placeholder after the image fails to load | Fire `error` on the `Image` | Placeholder `🖼️` is rendered | FR-CAT-01 | `src/components/__tests__/ProductImage.test.tsx` |
| TC-CT-IMG-03 | Component | `ProductImage` | Recovers the `Image` when the source changes after a failure | Fire `error`, then rerender with a new source | Placeholder gone; `Image` has the new source | FR-CAT-01 | `src/components/__tests__/ProductImage.test.tsx` |

### 5.8 Screen/Rendering — `CartScreen`

| Test Case ID | Type | Target | Scenario | Input / Action | Expected Result | Requirement Reference | Source |
|---|---|---|---|---|---|---|---|
| TC-SC-CART-01 | Screen/Rendering | `CartScreen` | Renders the empty state when the cart has no items | Render inside an empty `CartProvider` | `"Your cart is empty"` and `"Continue Shopping"` are shown | FR-CART-01 | `src/screens/__tests__/CartScreen.test.tsx` |
| TC-SC-CART-02 | Screen/Rendering | `CartScreen` | Navigates to Home when Continue Shopping is pressed | Press the `Continue shopping` control | `navigation.navigate('Home')` is called | FR-CART-01 | `src/screens/__tests__/CartScreen.test.tsx` |

## 6. Verified Test Result

The result below is the verified outcome for the captured frontend revision. It is not a permanent
project-wide guarantee and will change as the test suite evolves.

| Field | Value |
|---|---|
| Frontend commit | `36e02a6` |
| Merge commit | `866897b` |
| Test suites | 8 passed / 8 total |
| Tests | 45 passed / 45 total |
| Failed | 0 |
| Skipped | 0 |
| Snapshots | 0 |

Execution command: `npm test`.

## 7. Coverage

Coverage is point-in-time evidence generated by `npm run test:coverage` and tied to the frontend
revision in section 6. The `82.82%` overall statement figure below is **not** a permanent
project-wide quality guarantee.

### Overall

| Metric | Percentage |
|---|---|
| Statements | 82.82% |
| Branches | 65.82% |
| Functions | 73.58% |
| Lines | 81.72% |

### Measured modules

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

## 8. Requirement Traceability

Only requirement IDs that exist in the SRS (`02-requirements-specification-srs/`) are used. Where no
legitimate reference exists, tests are marked `No verified requirement reference`.

| Requirement ID | Requirement | Frontend tests |
|---|---|---|
| FR-CART-01 | A customer shall be able to view the cart. | TC-SC-CART-01, TC-SC-CART-02, TC-CTX-CART-01 |
| FR-CART-02 | A customer shall be able to add, update, and remove cart items. | TC-CTX-CART-02, TC-CTX-CART-03, TC-CTX-CART-06, TC-CTX-CART-07, TC-CTX-CART-08, TC-CTX-CART-09, TC-CTX-CART-10, TC-CTX-CART-11, TC-CTX-CART-12 |
| FR-CART-03 | A customer cart shall contain at most one row for a product. | TC-CTX-CART-04, TC-CTX-CART-05 |
| FR-CAT-03 | Customers shall be able to view a product and categories. | TC-UT-MAP-01, TC-UT-MAP-02, TC-UT-MAP-06, TC-UT-MAP-07, TC-UT-MAP-08, TC-CT-PRC-01, TC-CT-PRC-02, TC-CT-PRC-03, TC-CT-PRC-04 |
| FR-CAT-01 | Customers shall be able to browse products. | TC-UT-MAP-03, TC-UT-MAP-04, TC-UT-MAP-05, TC-CT-IMG-01, TC-CT-IMG-02, TC-CT-IMG-03 |

Notes:

- The `formatVND` utility tests, the `queryKeys` tests, and the generic `Button` tests are **not**
  mapped to any requirement; no legitimate reference exists, so none is invented.
- FR-CART-04 and FR-CART-05 (server-side stock validation and server-side price recalculation) are
  **not** covered by these frontend automated tests; those rules live in the backend. The backend
  now has a Cart API and separate tests, which do not establish frontend integration coverage.

### Explicitly not covered

| Requirement ID | Status |
|---|---|
| NFR-SEC-03 | Not covered by current frontend automated tests. |

## 9. Limitations and Future Testing

The current automated frontend tests do **not** verify:

- real backend integration;
- server-side Cart behavior;
- checkout;
- orders;
- payment / VNPAY;
- shipment;
- reviews;
- voucher;
- chat;
- AI;
- notifications;
- the Seller Product Management approval flow.

These features are not implemented or tested in the current frontend milestone and must not be
described as complete.

Known additional gaps inside the tested scope:

- `CartScreen` is covered only for its empty state; the item list, quantity controls, summary, and
  checkout placeholder are not tested.
- `CartItemRow` is not directly tested.
- `queryKeys` branch coverage is `0%` in the captured run; the assertions cover key shape, not every
  branch.

## 10. Evidence

The `04-testing/evidence/` directory is reserved for manually captured evidence. No screenshot is
committed yet; the files listed below do **not** currently exist.

Evidence to capture later:

| Evidence | Suggested filename | Captured from |
|---|---|---|
| Jest PASS summary | `evidence/jest-pass-<commit>.png` | `npm test` output |
| Coverage summary | `evidence/jest-coverage-<commit>.png` | `npm run test:coverage` |
| Typecheck pass | `evidence/typecheck-pass-<commit>.png` | `npm run typecheck` |
| Optional app behavior | `evidence/app-cart-empty-<commit>.png` | Running app, empty cart |

Each captured item should be labelled with the frontend commit and date it was produced.
