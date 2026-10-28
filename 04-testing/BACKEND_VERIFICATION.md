# Backend verification and requirements traceability

## Baseline and result

- Backend HEAD `2f34fef`; checked on 2026-10-28 on Windows, Java 21.0.8. The working-tree state at
  execution was not fully recorded, so this is not claimed to be a clean-commit run.
- Run from `backend`: `mvn test` — **BUILD SUCCESS, 231 tests, 0 failures, 0 errors,
  0 skipped**. This result applies only to this run and revision.
- Maven Surefire writes reports to `backend/target/surefire-reports/` on the execution machine;
  this build output is not evidence committed to the docs repository. Coverage was not measured.
- The test profile uses in-memory H2 with Flyway disabled; many API tests use MockMvc. Tests may
  use mocks; this result does not certify real PostgreSQL, Redis, VNPAY, or AI provider behavior.
- The Postman E2E journeys described in `backend/e2e/README.md` target local real HTTP, PostgreSQL
  and Flyway; **they were not run in this verification**. The collection creates real data and is
  local-only: do not change its base URL to Render. Real VNPAY sandbox, email and scheduled jobs are
  outside that collection.

## Selected traceability and test specification

IDs come from the [SRS](../02-requirements-specification-srs/README.md). This is a **partial** matrix,
not acceptance of all requirements. Class/method paths are under `backend/src/test/java/com/marketplace/`.
The rows passed in the `mvn test` run above; this is a test-level result, not full requirement or
frontend acceptance. The SRS and contracts remain the authoritative specifications.

| TC | Requirement | Setup and action | Expected result | Backend test |
|---|---|---|---|---|
| BE-ORD-01 | FR-ORD-01, FR-ORD-08 | Cart contains lines from two sellers; checkout selected lines. | One order contains the selected lines. | `controller/OrderApiTest#checkoutOfTwoSellersCreatesOneOrderFromTheSelectedLinesOnly` |
| BE-ORD-02 | FR-CART-04, FR-ORD-04 | One line has insufficient stock at checkout. | 409; no order or stock deduction for the valid line. | `controller/OrderApiTest#insufficientStockRejectsTheWholeCheckout` |
| BE-ORD-03 | FR-ORD-09 | Customer requests another customer's order. | 404; no order disclosed. | `controller/OrderApiTest#anotherCustomersOrderIs404` |
| BE-PAY-01 | FR-PAY-05 | Send a successful IPN and repeat it. | Order is paid; repeat callback changes nothing. | `controller/PaymentApiTest#aSuccessfulIpnMarksTheOrderPaidAndARepeatChangesNothing` |
| BE-SHP-01 | FR-ORD-13 | Customer confirms receipt of a DELIVERED order. | Order becomes COMPLETED. | `controller/ShipmentApiTest#theCustomerClosesADeliveredOrderByConfirmingReceipt` |
| BE-VCH-01 | FR-VCH-05 | Checkout with a valid voucher. | Discount applies and one use is consumed. | `controller/VoucherApiTest#checkoutAppliesThePercentageAndConsumesOneUse` |
| BE-AI-01 | FR-AI-10 | Guest asks the assistant. | An answer is returned, no session ID is returned and the session count does not increase. This test does not directly check message count. | `controller/AiApiTest#aGuestGetsAnAiAnswerAndNothingIsStored` |

Priority gaps: run local Postman E2E and retain results; verify real VNPAY sandbox, real email,
Flyway/PostgreSQL, scheduled jobs and load; test concurrent stock/voucher use on PostgreSQL.
`theLastItemCanBeSoldOnlyOnce` executes sequentially and is not a concurrent race test. Remaining
FR/NFR traceability, frontend integration and UAT **are not complete in this matrix**.

## Feature scope at the backend baseline

According to `backend/README.md`, the backend has auth/user, catalog, cart/order, COD/VNPAY sandbox,
shipment, voucher/review, AI and optional Redis cache. Human chat, RAG and Google login are not yet
implemented. Backend H2 tests were run as above; E2E was not run this time. Mobile and actual deployment
status were not checked in this verification; see the [frontend snapshot](README.md) and verify separately
before handover. Use cases and diagrams for unimplemented features are planned design, not run evidence.
