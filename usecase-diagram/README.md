# Use Case Diagrams

This folder contains editable draw.io use-case diagrams for the marketplace actors.

## Files

| File | Actor | Scope |
|---|---|---|
| `guest-usecases.drawio` | UNAUTHENTICATED USER | Public catalog, product details, registration, login, and guest AI product search |
| `customer-usecases.drawio` | CUSTOMER | Public catalog/search/details, cart, checkout, payment, orders, shipments, reviews, chat, and customer AI search |
| `seller-usecases.drawio` | SELLER | Owned products, stock, order items, shipments, and customer chat |
| `admin-usecases.drawio` | ADMIN | Users, categories, vouchers, moderation, monitoring, delivery configuration, and refunds |

## Notation rules

- Solid line: association, actor participates in a use case. Only manager and
  standalone use cases connect to the actor; CRUD leaves connect via `«extend»`.
- Dashed arrow labelled `«include»`: the source use case requires the target capability.
- In this project, `«include» Login` is an authentication precondition shorthand; it does not mean
  that an already-authenticated user logs in again for every request.
- `«include» Login` is drawn from each manager / standalone authenticated use case.
  CRUD leaves are covered transitively through their `«extend»` to the manager.
- Dashed arrow labelled `«extend»`: points from the extension (CRUD leaf) to the
  base manager use case, per UML; it shows which CRUD operations belong to each manager.
- Public catalog and public AI search use cases intentionally do not include `Login`.

## Style guide (matches `usecase-admin_new` sample)

- System boundary: flat `#f8fafc` rectangle, square corners, 1px `#334155` border,
  title centered on top (`Guest/Customer/Seller/Admin Use-Case`, 16pt).
- Actor: standard UML actor below the boundary, 1.5px stroke, 14pt regular label.
- Use case: ellipse 60px high; manager / `Login` filled `#79CFF4` with black 2px stroke;
  leaves and standalone use cases white with `#333333` 2px stroke; 14pt centered text.
- Associations: solid 1.5px `#334155`, routed orthogonally through a shared backbone
  below the use cases and side spines (no line crosses a use-case ellipse).
- `«include»` / `«extend»`: gray `#475569` dashed `5 5` arrows with filled heads,
  12pt labels, routed through vertical bus channels between columns.
- Open in draw.io via File &gt; Open; page sizes are set per diagram, export at 100%.

The diagrams describe planned system behavior. Deferred UI features remain visible only when they are
part of the backend role responsibility; the MVP boundary is documented in `PRD.md` and the SRS.
