# Documentation Guide

This directory contains the planning, requirements, architecture, data model, system overview, and
feature-level documentation for the E-Commerce Marketplace.

The documentation is based on:

- [`PLAN.md`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/PLAN.md), which
  is the implementation and scope baseline;
- [`ECommerce_Marketplace_Class_Diagram.puml`](https://github.com/anhhoang19122005/E-Commercial-Market-Place/blob/main/ECommerce_Marketplace_Class_Diagram.puml), which
  is the domain class-model reference.

The Mermaid diagrams in this directory are documentation views derived from those sources. They are
not a replacement for database migrations or Java implementation code.

## Recommended reading order

### 1. Project and delivery plan

Read [`01-project-overview-and-sdp/README.md`](01-project-overview-and-sdp/README.md) first. It
explains project scope, roles, technology baseline, development phases, quality gates, and Definition
of Done.

### 2. Requirements

Read [`02-requirements-specification-srs/README.md`](02-requirements-specification-srs/README.md) to
understand functional requirements, non-functional requirements, business rules, API boundaries,
security, and acceptance scenarios.

### 3. Architecture and design

Read [`03-architecture-and-design-specification/README.md`](03-architecture-and-design-specification/README.md)
for the modular-monolith structure, feature boundaries, workflows, AI design, persistence plan, and
security design.

### 4. Root reference documents

| Document | Purpose |
|---|---|
| [`PRD.md`](PRD.md) | Product scope, actors, capabilities, business goals, and product acceptance boundary |
| [`API_CONTRACT.md`](API_CONTRACT.md) | MVP request/response DTOs, query parameters, errors, authentication, and state rules |
| [`SYSTEM_OVERVIEW.md`](SYSTEM_OVERVIEW.md) | Runtime components and communication between mobile, backend, database, VNPAY, and LLM provider |
| [`ERD.md`](ERD.md) | Logical data relationships and persistence constraints |
| [`CLASS_DIAGRAM.md`](CLASS_DIAGRAM.md) | Full domain class diagram in Mermaid, based on the PlantUML reference |

### 5. Feature documentation

After reading the root documents, open the feature that you are implementing:

| Feature | Documentation |
|---|---|
| Auth | [`features/auth/README.md`](features/auth/README.md) |
| User | [`features/user/README.md`](features/user/README.md) |
| Catalog | [`features/catalog/README.md`](features/catalog/README.md) |
| Cart | [`features/cart/README.md`](features/cart/README.md) |
| Order | [`features/order/README.md`](features/order/README.md) |
| Payment | [`features/payment/README.md`](features/payment/README.md) |
| Shipment | [`features/shipment/README.md`](features/shipment/README.md) |
| Voucher | [`features/voucher/README.md`](features/voucher/README.md) |
| Review | [`features/review/README.md`](features/review/README.md) |
| Human chat | [`features/chat/README.md`](features/chat/README.md) |
| AI | [`features/ai/README.md`](features/ai/README.md) |
| Common | [`features/common/README.md`](features/common/README.md) |

Each feature README explains its responsibility, allowed actors, ownership rules, planned API area,
table relationships, field meanings, and a focused Mermaid class diagram. Use the focused diagram and
data-model section when implementing one feature so that you do not need to keep the complete domain
diagram in view at all times.

### 6. Use-case diagrams

Open [`usecase-diagram/README.md`](usecase-diagram/README.md) for editable draw.io diagrams separated
by `UNAUTHENTICATED USER`, `CUSTOMER`, `SELLER`, and `ADMIN`. Use these diagrams to check actor
permissions and the required `<<include>> Login` and `<<extend>>` relationships before implementing
an endpoint or screen.

### 7. Activity diagrams

Open [`activity-diagram/README.md`](activity-diagram/README.md) for one PlantUML activity diagram per
use case in `usecase-specification/`, grouped by actor. Use them to walk a flow step by step —
including validation, alternative, and exception paths — before implementing or reviewing an endpoint
or screen.

### 8. Sequence diagrams

Open [`sequence-diagram/README.md`](sequence-diagram/README.md) for a technical sequence diagram per
use case. These Mermaid diagrams trace the planned request through the feature controller, service,
repository, and database; feature-specific provider calls are included only when supported by the
contract. The layers are a target design and do not imply that
the current backend scaffold already implements them.

### 9. Testing

Read [`04-testing/README.md`](04-testing/README.md) for the automated frontend test suite: test
strategy, the full test case specification, the verified result, coverage, and requirement
traceability. This document covers implemented frontend automated tests only; backend integration,
system, and acceptance tests remain planned.

## Frontend-to-backend feature mapping

The React Native scaffold separates user journeys where the backend groups related business rules:

| Frontend feature | Backend documentation boundary |
|---|---|
| `checkout` | `order` + `payment` |
| `orders` | `order` + `shipment` + payment state display |
| `catalog` | `catalog` |
| `ai` | `ai` + public `catalog` search |

This mapping prevents the frontend folders from being mistaken for additional backend bounded
contexts.

## How to use the documents while coding

1. Confirm whether the feature is in the first mobile MVP in `PRD.md` and the SRS.
2. Confirm the request/response shape in [`API_CONTRACT.md`](API_CONTRACT.md).
3. Check the actor and ownership rules before adding an endpoint or service method.
4. Check `ERD.md` before adding or changing persistence relationships.
5. Check `CLASS_DIAGRAM.md` and the feature sub-diagram before adding a domain class or field.
6. Check `SYSTEM_OVERVIEW.md` before introducing a new external call or cross-feature dependency.
7. Update the relevant feature README when the agreed design changes.
8. For multi-seller delivery, read each `ShipmentStatus`; do not infer seller outcomes from the
   order-level `OrderStatus`.

## Naming note

The guide filename is `README.md`; its content uses normal Markdown syntax for repository preview
tools.
