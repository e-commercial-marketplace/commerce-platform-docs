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
| [`guest/register-customer-account.puml`](guest/register-customer-account.puml) | Register Customer Account | [`register-customer-account.vi.md`](../usecase-specification/guest/register-customer-account.vi.md) |
| [`guest/login.puml`](guest/login.puml) | Guest Login | [`login.vi.md`](../usecase-specification/guest/login.vi.md) |
| [`guest/login-with-google.puml`](guest/login-with-google.puml) | Guest Login with Google | [`login-with-google.vi.md`](../usecase-specification/guest/login-with-google.vi.md) |
| [`guest/browse-public-catalog.puml`](guest/browse-public-catalog.puml) | Browse Public Catalog | [`browse-public-catalog.vi.md`](../usecase-specification/guest/browse-public-catalog.vi.md) |
| [`guest/search-and-filter-products.puml`](guest/search-and-filter-products.puml) | Search and Filter Products | [`search-and-filter-products.vi.md`](../usecase-specification/guest/search-and-filter-products.vi.md) |
| [`guest/view-product-details.puml`](guest/view-product-details.puml) | View Product Details | [`view-product-details.vi.md`](../usecase-specification/guest/view-product-details.vi.md) |
| [`guest/use-ai-product-search.puml`](guest/use-ai-product-search.puml) | Search Products with AI Assistant | [`use-ai-product-search.vi.md`](../usecase-specification/guest/use-ai-product-search.vi.md) |

### Customer

| File | Use case | Specification |
|---|---|---|
| [`customer/login.puml`](customer/login.puml) | Customer Login | [`login.vi.md`](../usecase-specification/customer/login.vi.md) |
| [`customer/logout.puml`](customer/logout.puml) | Logout | [`logout.vi.md`](../usecase-specification/customer/logout.vi.md) |
| [`customer/browse-catalog.puml`](customer/browse-catalog.puml) | Browse Catalog | [`browse-catalog.vi.md`](../usecase-specification/customer/browse-catalog.vi.md) |
| [`customer/search-and-filter-products.puml`](customer/search-and-filter-products.puml) | Search and Filter Products | [`search-and-filter-products.vi.md`](../usecase-specification/customer/search-and-filter-products.vi.md) |
| [`customer/view-product-details.puml`](customer/view-product-details.puml) | View Product Details | [`view-product-details.vi.md`](../usecase-specification/customer/view-product-details.vi.md) |
| [`customer/use-ai-product-search.puml`](customer/use-ai-product-search.puml) | Search Products with AI Assistant | [`use-ai-product-search.vi.md`](../usecase-specification/customer/use-ai-product-search.vi.md) |
| [`customer/view-cart.puml`](customer/view-cart.puml) | View Cart | [`view-cart.vi.md`](../usecase-specification/customer/view-cart.vi.md) |
| [`customer/add-cart-item.puml`](customer/add-cart-item.puml) | Add Cart Item | [`add-cart-item.vi.md`](../usecase-specification/customer/add-cart-item.vi.md) |
| [`customer/update-cart-item.puml`](customer/update-cart-item.puml) | Update Cart Item | [`update-cart-item.vi.md`](../usecase-specification/customer/update-cart-item.vi.md) |
| [`customer/remove-cart-item.puml`](customer/remove-cart-item.puml) | Remove Cart Item | [`remove-cart-item.vi.md`](../usecase-specification/customer/remove-cart-item.vi.md) |
| [`customer/manage-cart.puml`](customer/manage-cart.puml) | Manage Cart | [`manage-cart.vi.md`](../usecase-specification/customer/manage-cart.vi.md) |
| [`customer/checkout-order.puml`](customer/checkout-order.puml) | Checkout Order | [`checkout-order.vi.md`](../usecase-specification/customer/checkout-order.vi.md) |
| [`customer/pay-for-order-cod.puml`](customer/pay-for-order-cod.puml) | Pay for Order (COD) | [`pay-for-order-cod.vi.md`](../usecase-specification/customer/pay-for-order-cod.vi.md) |
| [`customer/pay-for-order-vnpay.puml`](customer/pay-for-order-vnpay.puml) | Pay for Order (VNPAY) | [`pay-for-order-vnpay.vi.md`](../usecase-specification/customer/pay-for-order-vnpay.vi.md) |
| [`customer/view-my-orders.puml`](customer/view-my-orders.puml) | View My Orders | [`view-my-orders.vi.md`](../usecase-specification/customer/view-my-orders.vi.md) |
| [`customer/view-my-shipments.puml`](customer/view-my-shipments.puml) | View My Shipments | [`view-my-shipments.vi.md`](../usecase-specification/customer/view-my-shipments.vi.md) |
| [`customer/create-product-review.puml`](customer/create-product-review.puml) | Create Product Review | [`create-product-review.vi.md`](../usecase-specification/customer/create-product-review.vi.md) |
| [`customer/chat-with-seller.puml`](customer/chat-with-seller.puml) | Chat with Seller | [`chat-with-seller.vi.md`](../usecase-specification/customer/chat-with-seller.vi.md) |

### Seller

| File | Use case | Specification |
|---|---|---|
| [`seller/login.puml`](seller/login.puml) | Seller Login | [`login.vi.md`](../usecase-specification/seller/login.vi.md) |
| [`seller/manage-products.puml`](seller/manage-products.puml) | Manage Products | [`manage-products.vi.md`](../usecase-specification/seller/manage-products.vi.md) |
| [`seller/create-product.puml`](seller/create-product.puml) | Create Product | [`create-product.vi.md`](../usecase-specification/seller/create-product.vi.md) |
| [`seller/update-product.puml`](seller/update-product.puml) | Update Product | [`update-product.vi.md`](../usecase-specification/seller/update-product.vi.md) |
| [`seller/update-product-stock.puml`](seller/update-product-stock.puml) | Update Product Stock | [`update-product-stock.vi.md`](../usecase-specification/seller/update-product-stock.vi.md) |
| [`seller/delete-or-unlist-product.puml`](seller/delete-or-unlist-product.puml) | Delete or Unlist Product | [`delete-or-unlist-product.vi.md`](../usecase-specification/seller/delete-or-unlist-product.vi.md) |
| [`seller/view-own-products.puml`](seller/view-own-products.puml) | View Own Products | [`view-own-products.vi.md`](../usecase-specification/seller/view-own-products.vi.md) |
| [`seller/view-own-order-items.puml`](seller/view-own-order-items.puml) | View Own Order Items | [`view-own-order-items.vi.md`](../usecase-specification/seller/view-own-order-items.vi.md) |
| [`seller/confirm-order-item.puml`](seller/confirm-order-item.puml) | Confirm Order Item | [`confirm-order-item.vi.md`](../usecase-specification/seller/confirm-order-item.vi.md) |
| [`seller/cancel-order-item.puml`](seller/cancel-order-item.puml) | Cancel Order Item | [`cancel-order-item.vi.md`](../usecase-specification/seller/cancel-order-item.vi.md) |
| [`seller/process-own-order-items.puml`](seller/process-own-order-items.puml) | Process Own Order Items | [`process-own-order-items.vi.md`](../usecase-specification/seller/process-own-order-items.vi.md) |
| [`seller/choose-delivery-method.puml`](seller/choose-delivery-method.puml) | Choose Delivery Method | [`choose-delivery-method.vi.md`](../usecase-specification/seller/choose-delivery-method.vi.md) |
| [`seller/create-shipment.puml`](seller/create-shipment.puml) | Create Shipment | [`create-shipment.vi.md`](../usecase-specification/seller/create-shipment.vi.md) |
| [`seller/update-shipment-status.puml`](seller/update-shipment-status.puml) | Update Shipment Status | [`update-shipment-status.vi.md`](../usecase-specification/seller/update-shipment-status.vi.md) |
| [`seller/view-own-shipments.puml`](seller/view-own-shipments.puml) | View Own Shipments | [`view-own-shipments.vi.md`](../usecase-specification/seller/view-own-shipments.vi.md) |
| [`seller/manage-own-shipments.puml`](seller/manage-own-shipments.puml) | Manage Own Shipments | [`manage-own-shipments.vi.md`](../usecase-specification/seller/manage-own-shipments.vi.md) |
| [`seller/chat-with-customer.puml`](seller/chat-with-customer.puml) | Chat with Customer | [`chat-with-customer.vi.md`](../usecase-specification/seller/chat-with-customer.vi.md) |

### Admin

| File | Use case | Specification |
|---|---|---|
| [`admin/login.puml`](admin/login.puml) | Admin Login | [`login.vi.md`](../usecase-specification/admin/login.vi.md) |
| [`admin/manage-users.puml`](admin/manage-users.puml) | Manage Users | [`manage-users.vi.md`](../usecase-specification/admin/manage-users.vi.md) |
| [`admin/create-user.puml`](admin/create-user.puml) | Create User | [`create-user.vi.md`](../usecase-specification/admin/create-user.vi.md) |
| [`admin/update-user.puml`](admin/update-user.puml) | Update User | [`update-user.vi.md`](../usecase-specification/admin/update-user.vi.md) |
| [`admin/disable-or-delete-user.puml`](admin/disable-or-delete-user.puml) | Disable or Delete User | [`disable-or-delete-user.vi.md`](../usecase-specification/admin/disable-or-delete-user.vi.md) |
| [`admin/view-users.puml`](admin/view-users.puml) | View Users | [`view-users.vi.md`](../usecase-specification/admin/view-users.vi.md) |
| [`admin/approve-or-suspend-seller.puml`](admin/approve-or-suspend-seller.puml) | Approve or Suspend Seller | [`approve-or-suspend-seller.vi.md`](../usecase-specification/admin/approve-or-suspend-seller.vi.md) |
| [`admin/manage-categories.puml`](admin/manage-categories.puml) | Manage Categories | [`manage-categories.vi.md`](../usecase-specification/admin/manage-categories.vi.md) |
| [`admin/create-category.puml`](admin/create-category.puml) | Create Category | [`create-category.vi.md`](../usecase-specification/admin/create-category.vi.md) |
| [`admin/update-category.puml`](admin/update-category.puml) | Update Category | [`update-category.vi.md`](../usecase-specification/admin/update-category.vi.md) |
| [`admin/delete-category.puml`](admin/delete-category.puml) | Delete Category | [`delete-category.vi.md`](../usecase-specification/admin/delete-category.vi.md) |
| [`admin/view-categories.puml`](admin/view-categories.puml) | View Categories | [`view-categories.vi.md`](../usecase-specification/admin/view-categories.vi.md) |
| [`admin/manage-vouchers.puml`](admin/manage-vouchers.puml) | Manage Vouchers | [`manage-vouchers.vi.md`](../usecase-specification/admin/manage-vouchers.vi.md) |
| [`admin/create-voucher.puml`](admin/create-voucher.puml) | Create Voucher | [`create-voucher.vi.md`](../usecase-specification/admin/create-voucher.vi.md) |
| [`admin/update-voucher.puml`](admin/update-voucher.puml) | Update Voucher | [`update-voucher.vi.md`](../usecase-specification/admin/update-voucher.vi.md) |
| [`admin/delete-or-expire-voucher.puml`](admin/delete-or-expire-voucher.puml) | Delete or Expire Voucher | [`delete-or-expire-voucher.vi.md`](../usecase-specification/admin/delete-or-expire-voucher.vi.md) |
| [`admin/view-vouchers.puml`](admin/view-vouchers.puml) | View Vouchers | [`view-vouchers.vi.md`](../usecase-specification/admin/view-vouchers.vi.md) |
| [`admin/moderate-reviews-and-violations.puml`](admin/moderate-reviews-and-violations.puml) | Moderate Reviews and Violations | [`moderate-reviews-and-violations.vi.md`](../usecase-specification/admin/moderate-reviews-and-violations.vi.md) |
| [`admin/approve-refund.puml`](admin/approve-refund.puml) | Approve Refund | [`approve-refund.vi.md`](../usecase-specification/admin/approve-refund.vi.md) |
| [`admin/configure-platform-delivery.puml`](admin/configure-platform-delivery.puml) | Configure Platform Delivery | [`configure-platform-delivery.vi.md`](../usecase-specification/admin/configure-platform-delivery.vi.md) |
| [`admin/monitor-marketplace-operations.puml`](admin/monitor-marketplace-operations.puml) | Monitor Marketplace Operations | [`monitor-marketplace-operations.vi.md`](../usecase-specification/admin/monitor-marketplace-operations.vi.md) |

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
