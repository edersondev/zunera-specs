# Tasks: Expandable Statement Transactions

**Input**: [spec.md](spec.md), [plan.md](plan.md), [contract](contracts/statement-installment-response.yaml)
**Order**: Backend contract and tests before frontend implementation. All three repositories use `012-statement-transactions-expandable-list`.

## Phase 1: Setup

- [x] T001 Create matching feature branches and specification in `zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

## Phase 2: Foundational backend contract

- [x] T002 [US1] Add read-only purchase fields to `../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementDetailResource.php`.
- [x] T003 [US1] Assert purchase metadata, ownership and unchanged amounts in `../zunera-backend/tests/Feature/CreditCards/CreditCardStatementTest.php`.
- [x] T004 [US1] Run focused backend tests and Pint in `../zunera-backend` before frontend work.

## Phase 3: User Story 1 — Scan and inspect (P1)

**Independent test**: Open a multi-item statement, inspect all amounts, toggle one row by pointer and keyboard, then open another.

- [x] T005 [US1] Build the focused row presentation in `../zunera-frontend/src/components/credit-cards/ExpandableTransactionItem.vue`.
- [x] T006 [US1] Replace flat rows and own one expanded ID in `../zunera-frontend/src/components/credit-cards/CreditCardStatementLineItems.vue`.
- [x] T007 [US1] Pass statement card and billing context from `../zunera-frontend/src/views/credit-cards/CreditCardStatementView.vue`.
- [x] T008 [US1] Add Portuguese/English labels in `../zunera-frontend/src/i18n/messages.js`.
- [x] T009 [US1] Cover display, accessibility, responsive edge cases and disappearing IDs in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardStatementLineItems.spec.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.

## Phase 4: User Story 2 — Existing actions (P2)

**Independent test**: Open Correct or Refund from an installment, complete the current dialog, and see authoritative statement data refresh.

- [x] T010 [US2] Add a purchase lookup action through `../zunera-frontend/src/stores/credit-cards/creditCardStore.js` using the existing service.
- [x] T011 [US2] Wire current correction and credit-event dialogs in `../zunera-frontend/src/views/credit-cards/CreditCardStatementView.vue`.
- [x] T012 [US2] Cover action eligibility, non-toggle clicks, purchase lookup and refresh in `../zunera-frontend/src/views/credit-cards/__tests__/CreditCardStatementView.spec.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.

## Phase 5: Validation

- [x] T013 Run frontend unit tests, lint, build and isolated Playwright credit-card suite in `../zunera-frontend`.
- [x] T014 Confirm exact statement totals and finish checklist in `specs/012-statement-transactions-expandable-list/tasks.md`.

**Dependencies**: T002–T004 precede T005–T012; T010 precedes T011–T012. Other frontend tasks can run after T004.

## Validation record

- Backend Credit Card tests: 282 passed; focused statement tests: 6 passed. Full PHPUnit suite: 590 passed with a 512 MB limit (the Artisan wrapper's 128 MB child limit stopped an unrelated scale test). Pint formatted the two touched PHP files.
- Frontend unit tests: 419 passed. Lint and production build passed. No type-check script exists.
- Targeted statement Playwright checks: 9 passed across Chromium, Firefox and WebKit, including mobile dark mode, keyboard toggle and both purchase actions.
- Full Credit Cards Playwright suite: 39 passed, 9 failed across the same three older cases (purchase form over-limit flow, transaction-history locator, dashboard locale expectation). Those tests and application pages were not changed by this feature.
