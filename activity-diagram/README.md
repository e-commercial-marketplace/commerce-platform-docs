# Activity Diagrams

This folder contains one PlantUML activity diagram per use case documented in
[`usecase-specification/`](../usecase-specification/). Each diagram is the flow view of its spec:
main flow, alternative flows, exception flows, and the pre/postconditions, rendered with the Visual
Paradigm PlantUML theme and two swimlanes (the primary actor and the system).

The diagrams are documentation views. They do not replace the use-case specs, the API contract, or
the implementation code.

## Files

### Guest

| File | Use case | Specification |
|---|---|---|
| [`guest/browse-public-catalog.puml`](guest/browse-public-catalog.puml) | Browse Public Catalog | [`browse-public-catalog.vi.md`](../usecase-specification/guest/browse-public-catalog.vi.md) |
| [`guest/login.puml`](guest/login.puml) | Guest Login | [`login.vi.md`](../usecase-specification/guest/login.vi.md) |
| [`guest/register-account.puml`](guest/register-account.puml) | Register Account | [`register-account.vi.md`](../usecase-specification/guest/register-account.vi.md) |
| [`guest/reset-password.puml`](guest/reset-password.puml) | Reset Password | [`reset-password.vi.md`](../usecase-specification/guest/reset-password.vi.md) |
| [`guest/search-and-filter-products.puml`](guest/search-and-filter-products.puml) | Search and Filter Products | [`search-and-filter-products.vi.md`](../usecase-specification/guest/search-and-filter-products.vi.md) |
| [`guest/use-ai-product-search.puml`](guest/use-ai-product-search.puml) | Search Products with AI Assistant | [`use-ai-product-search.vi.md`](../usecase-specification/guest/use-ai-product-search.vi.md) |
| [`guest/view-product-details.puml`](guest/view-product-details.puml) | View Product Details | [`view-product-details.vi.md`](../usecase-specification/guest/view-product-details.vi.md) |
| [`guest/view-product-reviews.puml`](guest/view-product-reviews.puml) | View Product Reviews | [`view-product-reviews.vi.md`](../usecase-specification/guest/view-product-reviews.vi.md) |
| [`guest/view-seller-shop.puml`](guest/view-seller-shop.puml) | View Seller Shop | [`view-seller-shop.vi.md`](../usecase-specification/guest/view-seller-shop.vi.md) |

### Customer

| File | Use case | Specification |
|---|---|---|
| [`customer/add-cart-item.puml`](customer/add-cart-item.puml) | Add Cart Item | [`add-cart-item.vi.md`](../usecase-specification/customer/add-cart-item.vi.md) |
| [`customer/apply-voucher.puml`](customer/apply-voucher.puml) | Apply Voucher | [`apply-voucher.vi.md`](../usecase-specification/customer/apply-voucher.vi.md) |
| [`customer/browse-catalog.puml`](customer/browse-catalog.puml) | Browse Catalog | [`browse-catalog.vi.md`](../usecase-specification/customer/browse-catalog.vi.md) |
| [`customer/change-password.puml`](customer/change-password.puml) | Change Password | [`change-password.vi.md`](../usecase-specification/customer/change-password.vi.md) |
| [`customer/chat-with-seller.puml`](customer/chat-with-seller.puml) | Chat with Seller | [`chat-with-seller.vi.md`](../usecase-specification/customer/chat-with-seller.vi.md) |
| [`customer/checkout-order.puml`](customer/checkout-order.puml) | Checkout Order | [`checkout-order.vi.md`](../usecase-specification/customer/checkout-order.vi.md) |
| [`customer/confirm-order-received.puml`](customer/confirm-order-received.puml) | Confirm Order Received | [`confirm-order-received.vi.md`](../usecase-specification/customer/confirm-order-received.vi.md) |
| [`customer/create-product-review.puml`](customer/create-product-review.puml) | Create Product Review | [`create-product-review.vi.md`](../usecase-specification/customer/create-product-review.vi.md) |
| [`customer/login.puml`](customer/login.puml) | Customer Login | [`login.vi.md`](../usecase-specification/customer/login.vi.md) |
| [`customer/logout.puml`](customer/logout.puml) | Logout | [`logout.vi.md`](../usecase-specification/customer/logout.vi.md) |
| [`customer/manage-cart.puml`](customer/manage-cart.puml) | Manage Cart | [`manage-cart.vi.md`](../usecase-specification/customer/manage-cart.vi.md) |
| [`customer/manage-profile.puml`](customer/manage-profile.puml) | Manage Profile | [`manage-profile.vi.md`](../usecase-specification/customer/manage-profile.vi.md) |
| [`customer/pay-for-order-cod.puml`](customer/pay-for-order-cod.puml) | Pay for Order (COD) | [`pay-for-order-cod.vi.md`](../usecase-specification/customer/pay-for-order-cod.vi.md) |
| [`customer/pay-for-order-vnpay.puml`](customer/pay-for-order-vnpay.puml) | Pay for Order (VNPAY) | [`pay-for-order-vnpay.vi.md`](../usecase-specification/customer/pay-for-order-vnpay.vi.md) |
| [`customer/remove-cart-item.puml`](customer/remove-cart-item.puml) | Remove Cart Item | [`remove-cart-item.vi.md`](../usecase-specification/customer/remove-cart-item.vi.md) |
| [`customer/search-and-filter-products.puml`](customer/search-and-filter-products.puml) | Search and Filter Products | [`search-and-filter-products.vi.md`](../usecase-specification/customer/search-and-filter-products.vi.md) |
| [`customer/update-cart-item.puml`](customer/update-cart-item.puml) | Update Cart Item | [`update-cart-item.vi.md`](../usecase-specification/customer/update-cart-item.vi.md) |
| [`customer/upload-image.puml`](customer/upload-image.puml) | Upload Image | [`upload-image.vi.md`](../usecase-specification/customer/upload-image.vi.md) |
| [`customer/use-ai-product-search.puml`](customer/use-ai-product-search.puml) | Search Products with AI Assistant | [`use-ai-product-search.vi.md`](../usecase-specification/customer/use-ai-product-search.vi.md) |
| [`customer/view-ai-chat-history.puml`](customer/view-ai-chat-history.puml) | View AI Chat History | [`view-ai-chat-history.vi.md`](../usecase-specification/customer/view-ai-chat-history.vi.md) |
| [`customer/view-cart.puml`](customer/view-cart.puml) | View Cart | [`view-cart.vi.md`](../usecase-specification/customer/view-cart.vi.md) |
| [`customer/view-my-orders.puml`](customer/view-my-orders.puml) | View My Orders | [`view-my-orders.vi.md`](../usecase-specification/customer/view-my-orders.vi.md) |
| [`customer/view-my-shipments.puml`](customer/view-my-shipments.puml) | View My Shipments | [`view-my-shipments.vi.md`](../usecase-specification/customer/view-my-shipments.vi.md) |
| [`customer/view-product-details.puml`](customer/view-product-details.puml) | View Product Details | [`view-product-details.vi.md`](../usecase-specification/customer/view-product-details.vi.md) |
| [`customer/view-product-reviews.puml`](customer/view-product-reviews.puml) | View Product Reviews | [`view-product-reviews.vi.md`](../usecase-specification/customer/view-product-reviews.vi.md) |
| [`customer/view-seller-shop.puml`](customer/view-seller-shop.puml) | View Seller Shop | [`view-seller-shop.vi.md`](../usecase-specification/customer/view-seller-shop.vi.md) |

### Seller

| File | Use case | Specification |
|---|---|---|
| [`seller/cancel-order-item.puml`](seller/cancel-order-item.puml) | Cancel Order Item | [`cancel-order-item.vi.md`](../usecase-specification/seller/cancel-order-item.vi.md) |
| [`seller/chat-with-customer.puml`](seller/chat-with-customer.puml) | Chat with Customer | [`chat-with-customer.vi.md`](../usecase-specification/seller/chat-with-customer.vi.md) |
| [`seller/choose-delivery-method.puml`](seller/choose-delivery-method.puml) | Choose Delivery Method | [`choose-delivery-method.vi.md`](../usecase-specification/seller/choose-delivery-method.vi.md) |
| [`seller/confirm-order-item.puml`](seller/confirm-order-item.puml) | Confirm Order Item | [`confirm-order-item.vi.md`](../usecase-specification/seller/confirm-order-item.vi.md) |
| [`seller/create-product.puml`](seller/create-product.puml) | Create Product | [`create-product.vi.md`](../usecase-specification/seller/create-product.vi.md) |
| [`seller/create-shipment.puml`](seller/create-shipment.puml) | Create Shipment | [`create-shipment.vi.md`](../usecase-specification/seller/create-shipment.vi.md) |
| [`seller/login.puml`](seller/login.puml) | Seller Login | [`login.vi.md`](../usecase-specification/seller/login.vi.md) |
| [`seller/manage-own-shipments.puml`](seller/manage-own-shipments.puml) | Manage Own Shipments | [`manage-own-shipments.vi.md`](../usecase-specification/seller/manage-own-shipments.vi.md) |
| [`seller/manage-products.puml`](seller/manage-products.puml) | Manage Products | [`manage-products.vi.md`](../usecase-specification/seller/manage-products.vi.md) |
| [`seller/process-own-order-items.puml`](seller/process-own-order-items.puml) | Process Own Order Items | [`process-own-order-items.vi.md`](../usecase-specification/seller/process-own-order-items.vi.md) |
| [`seller/unlist-product.puml`](seller/unlist-product.puml) | Unlist Product | [`unlist-product.vi.md`](../usecase-specification/seller/unlist-product.vi.md) |
| [`seller/update-product.puml`](seller/update-product.puml) | Update Product | [`update-product.vi.md`](../usecase-specification/seller/update-product.vi.md) |
| [`seller/update-product-stock.puml`](seller/update-product-stock.puml) | Update Product Stock | [`update-product-stock.vi.md`](../usecase-specification/seller/update-product-stock.vi.md) |
| [`seller/update-shipment-status.puml`](seller/update-shipment-status.puml) | Update Shipment Status | [`update-shipment-status.vi.md`](../usecase-specification/seller/update-shipment-status.vi.md) |
| [`seller/upload-image.puml`](seller/upload-image.puml) | Upload Image | [`upload-image.vi.md`](../usecase-specification/seller/upload-image.vi.md) |
| [`seller/view-own-order-items.puml`](seller/view-own-order-items.puml) | View Own Order Items | [`view-own-order-items.vi.md`](../usecase-specification/seller/view-own-order-items.vi.md) |
| [`seller/view-own-products.puml`](seller/view-own-products.puml) | View Own Products | [`view-own-products.vi.md`](../usecase-specification/seller/view-own-products.vi.md) |
| [`seller/view-own-shipments.puml`](seller/view-own-shipments.puml) | View Own Shipments | [`view-own-shipments.vi.md`](../usecase-specification/seller/view-own-shipments.vi.md) |

### Admin

| File | Use case | Specification |
|---|---|---|
| [`admin/approve-or-suspend-seller.puml`](admin/approve-or-suspend-seller.puml) | Approve or Suspend Seller | [`approve-or-suspend-seller.vi.md`](../usecase-specification/admin/approve-or-suspend-seller.vi.md) |
| [`admin/create-category.puml`](admin/create-category.puml) | Create Category | [`create-category.vi.md`](../usecase-specification/admin/create-category.vi.md) |
| [`admin/create-user.puml`](admin/create-user.puml) | Create User | [`create-user.vi.md`](../usecase-specification/admin/create-user.vi.md) |
| [`admin/create-voucher.puml`](admin/create-voucher.puml) | Create Voucher | [`create-voucher.vi.md`](../usecase-specification/admin/create-voucher.vi.md) |
| [`admin/delete-category.puml`](admin/delete-category.puml) | Delete Category | [`delete-category.vi.md`](../usecase-specification/admin/delete-category.vi.md) |
| [`admin/disable-user.puml`](admin/disable-user.puml) | Disable User | [`disable-user.vi.md`](../usecase-specification/admin/disable-user.vi.md) |
| [`admin/expire-voucher.puml`](admin/expire-voucher.puml) | Expire Voucher | [`expire-voucher.vi.md`](../usecase-specification/admin/expire-voucher.vi.md) |
| [`admin/login.puml`](admin/login.puml) | Admin Login | [`login.vi.md`](../usecase-specification/admin/login.vi.md) |
| [`admin/manage-carriers.puml`](admin/manage-carriers.puml) | Manage Carriers | [`manage-carriers.vi.md`](../usecase-specification/admin/manage-carriers.vi.md) |
| [`admin/manage-categories.puml`](admin/manage-categories.puml) | Manage Categories | [`manage-categories.vi.md`](../usecase-specification/admin/manage-categories.vi.md) |
| [`admin/manage-users.puml`](admin/manage-users.puml) | Manage Users | [`manage-users.vi.md`](../usecase-specification/admin/manage-users.vi.md) |
| [`admin/manage-vouchers.puml`](admin/manage-vouchers.puml) | Manage Vouchers | [`manage-vouchers.vi.md`](../usecase-specification/admin/manage-vouchers.vi.md) |
| [`admin/update-category.puml`](admin/update-category.puml) | Update Category | [`update-category.vi.md`](../usecase-specification/admin/update-category.vi.md) |
| [`admin/update-user.puml`](admin/update-user.puml) | Update User | [`update-user.vi.md`](../usecase-specification/admin/update-user.vi.md) |
| [`admin/update-voucher.puml`](admin/update-voucher.puml) | Update Voucher | [`update-voucher.vi.md`](../usecase-specification/admin/update-voucher.vi.md) |
| [`admin/view-categories.puml`](admin/view-categories.puml) | View Categories | [`view-categories.vi.md`](../usecase-specification/admin/view-categories.vi.md) |
| [`admin/view-users.puml`](admin/view-users.puml) | View Users | [`view-users.vi.md`](../usecase-specification/admin/view-users.vi.md) |
| [`admin/view-vouchers.puml`](admin/view-vouchers.puml) | View Vouchers | [`view-vouchers.vi.md`](../usecase-specification/admin/view-vouchers.vi.md) |

## Rendering

- PlantUML CLI: `java -jar plantuml.jar -tpng activity-diagram/guest/login.puml` (use `-tsvg` for
  SVG).
- VS Code: install a PlantUML extension and open the `.puml` file (`Alt+D` previews).
- Every file includes the theme
  `https://static.visual-paradigm.com/web/resources/plantuml-stdlib/themes/vp.puml`, so the first
  render needs network access; PlantUML caches the theme afterwards.
- Only the `.puml` sources are committed; generated PNG/SVG files are throwaway artifacts.

## Conventions

- Every file starts with the Visual Paradigm theme include and the same `skinparam` block: blue
  activities (`#7DD3FC` fill, `#2563EB` border), black start node, white end node, 1.5px `#222222`
  arrows and swimlane borders.
- Title format: `<area> - <use case>`, where `<area>` is one of Auth, Catalog, Cart, Order, Payment,
  Shipment, Voucher, Review, Chat, AI, User, Category, Admin.
- Swimlanes: the primary actor of the spec (`Guest`, `Customer`, `Seller`, or `Admin`) and `System`.
  No spec has a secondary actor. The four `login` use cases run in the `Guest` lane because their
  primary actor is a guest choosing an account to sign in with.
- The precondition and postcondition are a note pinned to the start node (PlantUML silently drops a
  floating note placed before `start`, so the note follows `start` instead).
- Main flow steps become activity nodes in the order of the spec table; each validation or business
  rule becomes a decision diamond with Yes/No branches.
- Recoverable alternative flows (the spec says the actor corrects the data and returns to a step) use
  a `repeat`/`repeat while` loop around the attempt, so the first attempt is not skipped. Put a line
  break in the retry label (for example, `Submit\nagain?`) so the loop connector passes between the
  text lines instead of through the label. Terminal failures ("kết thúc lần xử lý này") and
  rejections end in `stop`.
- Keep retry-decision labels concise; the actor is already clear from the swimlane, and long
  questions become cramped inside the decision diamond.
- Exception flows are terminal `stop` branches; the diagrams never invent a recovery the spec does
  not describe.
- Diagram text is English; the specs remain Vietnamese. Keep the two in sync when a spec changes.

Related documents: [`usecase-specification/`](../usecase-specification/) (source specs),
[`usecase-diagram/`](../usecase-diagram/README.md) (actor-level use-case diagrams), and the MVP
boundary in `PRD.md` and the SRS.
