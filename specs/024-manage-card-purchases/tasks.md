# Tasks: Manage Credit Card Purchases

**Input**: [spec.md](spec.md), [plan.md](plan.md), [research.md](research.md), [data-model.md](data-model.md), [API contract](contracts/purchase-statement-management-api.yaml), and [quickstart.md](quickstart.md)

**Tests**: Required by SQR-003 and the project tasks template. Write focused failing tests before changed behavior where practical; execute them before each backend API gate and frontend delivery gate.

**Delivery rule**: Finish all backend contract, authorization, validation, reconciliation, and focused tests through T040 before starting any frontend task. Every application checkout must remain on `024-manage-card-purchases`; preserve the unrelated frontend `package-lock.json` change. No new packages.

**Paths**: Backend paths begin `../zunera-backend/`; frontend paths begin `../zunera-frontend/`; specification paths begin `specs/024-manage-card-purchases/`.

## Phase 1: Setup

**Purpose**: Confirm the three coordinated checkouts before editing either application.

- [ ] T001 Verify `024-manage-card-purchases` in `specs/024-manage-card-purchases/plan.md`, `../zunera-backend/.git/HEAD`, and `../zunera-frontend/.git/HEAD`; record and preserve the existing `../zunera-frontend/package-lock.json` change.

---

## Phase 2: Foundational Backend (blocks every story)

**Purpose**: Give moves and deletion the same cycle rules and card-wide payment-surplus reconciliation.

- [ ] T002 [P] Add failing cycle-resolution tests for inclusive closing dates, month-end clamping, historical statement snapshots, and direct empty-future-month lookup in `../zunera-backend/tests/Unit/CreditCards/StatementCycleResolverTest.php`.
- [ ] T003 [P] Add failing reconciliation tests for paid source statements, surplus applied to oldest unpaid statements, residual card credit, payment update/remove/restore, and nonnegative outstanding in `../zunera-backend/tests/Unit/CreditCards/CreditCardObligationReconcilerTest.php`.
- [ ] T004 Create `credit_card_payment_surplus_applications` with owner/card/source/destination foreign keys, positive amount, active/superseded timestamps, and lookup indexes in `../zunera-backend/database/migrations/2026_10_20_000001_create_credit_card_payment_surplus_applications_table.php`.
- [ ] T005 Create the `CreditCardPaymentSurplusApplication` model with owned statement relationships and active allocation scope in `../zunera-backend/app/Models/CreditCardPaymentSurplusApplication.php`.
- [ ] T006 Implement canonical same-card cycle lookup, historical-row precedence, date eligibility, and finite window/direct-month options in `../zunera-backend/app/Services/CreditCards/StatementCycleResolver.php`, using `../zunera-backend/app/Services/CreditCards/BillingCycleCalculator.php`.
- [ ] T007 Reconcile all statement gross/net charges, effective payments, active credit allocations, surplus allocations, used credit, card credit, and available credit in centavos under deterministic card/statement locks in `../zunera-backend/app/Services/CreditCards/CreditCardObligationReconciler.php`.
- [ ] T008 Trigger the card-wide reconciliation after payment and credit-event lifecycle changes without changing paying-account movements in `../zunera-backend/app/Services/CreditCards/CreditCardStatementPaymentService.php` and `../zunera-backend/app/Services/CreditCards/CreditCardCreditEventService.php`.
- [ ] T009 Run the focused foundational tests and migration rollback/reapply, fixing defects in `../zunera-backend/tests/Unit/CreditCards/StatementCycleResolverTest.php`, `../zunera-backend/tests/Unit/CreditCards/CreditCardObligationReconcilerTest.php`, and `../zunera-backend/database/migrations/2026_10_20_000001_create_credit_card_payment_surplus_applications_table.php`.

**Checkpoint**: Cycle identity and payment surplus work before any purchase mutation is added.

---

## Phase 3: User Story 1 Backend — Move a Purchase Between Statements (P1)

**Goal**: Preview and move a full schedule or one installment, retaining payment history and a readable move audit.

**Independent Test**: Preview and confirm each mode on a two-installment purchase; verify one changed allocation per installment, consecutive full schedules, valid shared/gapped individual schedules, unchanged cash movements, traceable history, and same-key replay.

### Tests first

- [ ] T010 [P] [US1] Add failing owner-scoped contract tests for the preview, confirmation, and history response shapes, invalid mode/target/installment, stale preview, missing/invalid `Idempotency-Key`, and idempotent replay in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseMoveContractTest.php`.
- [ ] T011 [P] [US1] Add a deterministic 100-case matrix of distinct single/multi-installment moves across month/year boundaries with exact totals and one statement per installment; also test closed/paid sources, linked credit reallocation, full-schedule rebuilding, individual ordering, budgets, dashboard, history, and notifications in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseMoveAccountingTest.php`.

### Implementation

- [ ] T012 [P] [US1] Create the owner-scoped move-history table with purchase/installment cascade, source/destination statement snapshots, mode/time/operation identity, and unique replay guard in `../zunera-backend/database/migrations/2026_10_20_000002_create_credit_card_purchase_moves_table.php`.
- [ ] T013 [P] [US1] Add nullable `superseded_at` and active-application lookup index for historical credit allocations in `../zunera-backend/database/migrations/2026_10_20_000003_add_superseded_at_to_credit_card_credit_applications_table.php`.
- [ ] T014 [US1] Create move-history and active-credit relationships/scopes in `../zunera-backend/app/Models/CreditCardPurchaseMove.php`, `../zunera-backend/app/Models/CreditCardPurchase.php`, and `../zunera-backend/app/Models/CreditCardCreditApplication.php`.
- [ ] T015 [P] [US1] Define validated mode, target closing date, optional installment, preview token, and required `Idempotency-Key` header for confirmed moves in `../zunera-backend/app/Data/CreditCards/MovePurchaseData.php`, `../zunera-backend/app/Http/Requests/CreditCards/PreviewCreditCardPurchaseMoveRequest.php`, and `../zunera-backend/app/Http/Requests/CreditCards/MoveCreditCardPurchaseRequest.php`.
- [ ] T016 [US1] Build a read-only preview with source/destination periods, due dates, changed installments, resulting charges/outstanding/surplus, and a state fingerprint in `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseMoveService.php`.
- [ ] T017 [US1] Add locked, idempotent move execution to `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseMoveService.php`: reject stale previews, materialize eligible cycles once, preserve installment IDs/amounts, rebuild full schedules or move one ordered installment, and insert history only for changed rows.
- [ ] T018 [US1] Supersede and rebuild active linked credit-event applications after moves, with active-only statement and installment sums, in `../zunera-backend/app/Services/CreditCards/CreditCardCreditEventService.php` and `../zunera-backend/app/Services/CreditCards/CreditCardObligationReconciler.php`.
- [ ] T019 [US1] Render preview, confirmed move, and history through dedicated `../zunera-backend/app/Http/Resources/CreditCards/CreditCardPurchaseMovePreviewResource.php`, `../zunera-backend/app/Http/Resources/CreditCards/CreditCardPurchaseMoveResultResource.php`, and `../zunera-backend/app/Http/Resources/CreditCards/CreditCardPurchaseMoveResource.php`; expose them through `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardPurchaseController.php` and `../zunera-backend/routes/api.php` following `specs/024-manage-card-purchases/contracts/purchase-statement-management-api.yaml`.
- [ ] T020 [US1] Recompute recognition months, category spending, budget/dashboard/history summaries, and old/new statement notifications after a move in `../zunera-backend/app/Services/CreditCards/RecognizedCardExpenseProjection.php`, `../zunera-backend/app/Services/CreditCards/CreditCardBudgetProjectionService.php`, and `../zunera-backend/app/Services/Notifications/StatementNotificationProjector.php`.
- [ ] T021 [US1] Run focused backend move and authorization tests plus Pint; close every failing move-contract and monetary case in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseMoveContractTest.php` and `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseMoveAccountingTest.php` before frontend work.

**Checkpoint**: US1 backend is independently testable from the API with no Vue changes.

---

## Phase 4: User Story 2 Backend — Choose Statement for a Backdated Purchase (P1)

**Goal**: Require an explicit same-card statement choice only after the purchase date's derived cycle has closed.

**Independent Test**: Create purchases on open, closing, and closed dates; verify hidden/required choice rules, selected first cycle, consecutive later installments, future empty cycle, stale/foreign choice errors, and unchanged recurring generation.

### Tests first

- [ ] T022 [P] [US2] Add a deterministic 100-case before/on/after-closing date matrix for closed-cycle detection, required choice, and saved installment allocation; also test the statement-options response shape, owner isolation, São Paulo business day, invalid `purchase_date`/`from_month`/`limit`, window/direct-month lookup, no read-side materialization, and stale choice in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseStatementChoiceTest.php`.
- [ ] T023 [P] [US2] Add failing regression tests proving recurring occurrence purchase generation retains automatic date-based placement in `../zunera-backend/tests/Feature/CreditCards/RecurringCardPurchaseAccountingTest.php`.

### Implementation

- [ ] T024 [US2] Add conditional `statement_closing_date` validation and create DTO field, including the idempotency fingerprint and field-level errors, in `../zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardPurchaseRequest.php` and `../zunera-backend/app/Data/CreditCards/CreatePurchaseData.php`.
- [ ] T025 [US2] Allocate manual purchases from the explicit first cycle only when the date-derived cycle has closed, then generate consecutive installments; preserve automatic recurring allocation in `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php`.
- [ ] T026 [US2] Validate `purchase_date`, `from_month`, and `limit` in `../zunera-backend/app/Http/Requests/CreditCards/ListCreditCardStatementOptionsRequest.php`; render `requires_statement_choice`, derived closing date, and windowed period/closing/due-date options without materializing empty statements through `../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementOptionsResource.php` and `../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementOptionResource.php`, exposed by `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardPurchaseController.php` and `../zunera-backend/routes/api.php`.
- [ ] T027 [US2] Run statement-choice, recurring-regression, purchase-creation, and authorization tests plus Pint; align responses with `specs/024-manage-card-purchases/contracts/purchase-statement-management-api.yaml` and `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseStatementChoiceTest.php`.

**Checkpoint**: US2 backend is independently testable from the API with no Vue changes.

---

## Phase 5: User Story 3 Backend — Permanently Delete a Purchase (P2)

**Goal**: Remove purchase charges and installments permanently while keeping payments, linked-credit safety, recurrence identity, and replay protection.

**Independent Test**: Confirm a linked-credit-free purchase disappears from all views; payments remain and surplus reconciles; linked-credit deletion fails; repeated deletion has no second effect; deleted recurring occurrence cannot regenerate.

### Tests first

- [ ] T028 [P] [US3] Add failing owner-scoped DELETE contract tests for missing/invalid `Idempotency-Key`, permanent removal, any linked credit block, identical minimal receipt shape on initial success and same-key replay, new-key missing-resource response, and old create/update/move replay privacy in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseDeleteContractTest.php`.
- [ ] T029 [P] [US3] Add a deterministic 100-case move/delete matrix across open, closed, partly paid, and paid statements that checks every balance/projection and no duplicate cash movement; add a separate 100-attempt repeated/overlapping move/delete matrix proving at-most-once effects in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseDeleteAccountingTest.php`.
- [ ] T030 [P] [US3] Add failing recorded-occurrence deletion, marker visibility, replay/catch-up nonregeneration, and unaffected sibling tests in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardDeletedPurchaseTest.php`.

### Implementation

- [ ] T031 [US3] Add nullable `purchase_deleted_at` to recorded card occurrences in `../zunera-backend/database/migrations/2026_10_20_000004_add_purchase_deleted_at_to_recurring_card_occurrences_table.php`.
- [ ] T032 [US3] Persist and expose the recorded occurrence's deleted-purchase marker while keeping the occurrence non-generatable in `../zunera-backend/app/Models/RecurringCardOccurrence.php`, `../zunera-backend/app/Http/Resources/RecurringTransactions/CardOccurrenceResource.php`, and `../zunera-backend/app/Services/RecurringTransactions/RecurringCardOccurrenceActionService.php`.
- [ ] T033 [US3] Implement locked hard delete with owned purchase lookup, any-linked-credit conflict, installment/move cascade, retained statement/payment rows, affected-statement receipt, and card-wide reconciliation in `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseDeleteService.php`.
- [ ] T034 [US3] Retain claimed keys/fingerprints but scrub purchase-specific card and recurrence response snapshots; resolve identical DELETE replay before the missing-row lookup and reject old create/update/move replays safely in `../zunera-backend/app/Services/CreditCards/CreditCardMutationIdempotencyService.php`, `../zunera-backend/app/Services/RecurringTransactions/RecurringIdempotencyService.php`, `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php`, and `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseCorrectionService.php`.
- [ ] T035 [US3] Validate the DELETE `Idempotency-Key` header in `../zunera-backend/app/Http/Requests/CreditCards/DeleteCreditCardPurchaseRequest.php`; render the minimal receipt on first success and same-key replay through `../zunera-backend/app/Http/Resources/CreditCards/CreditCardPurchaseDeleteResource.php`; add the owner-scoped endpoint and linked-credit conflict in `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardPurchaseController.php`, `../zunera-backend/app/Exceptions/CreditCards/CreditCardStateException.php`, and `../zunera-backend/routes/api.php` following `specs/024-manage-card-purchases/contracts/purchase-statement-management-api.yaml`.
- [ ] T036 [US3] Refresh changed spending months, category/budget/dashboard/history totals, and affected statement notifications once after delete in `../zunera-backend/app/Services/CreditCards/RecognizedCardExpenseProjection.php`, `../zunera-backend/app/Services/CreditCards/CreditCardBudgetProjectionService.php`, and `../zunera-backend/app/Services/Notifications/StatementNotificationProjector.php`.
- [ ] T037 [US3] Run focused delete, recurrence, authorization, and payment-surplus tests plus Pint; close every failing case in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseDeleteContractTest.php`, `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseDeleteAccountingTest.php`, and `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardDeletedPurchaseTest.php`.

**Checkpoint**: US3 backend is independently testable from the API with no Vue changes.

---

## Phase 6: Backend Contract Gate

**Purpose**: Complete the project-required backend-first gate for all three stories.

- [ ] T038 Seed a representative card with 1,000 purchases; after 10 warmups, measure 100 statement-option requests and 100 move-preview requests separately, assert each p95 is under 2 seconds, and inspect bounded query plans in `../zunera-backend/tests/Feature/CreditCards/CreditCardQueryPerformanceTest.php` and `../zunera-backend/tests/Performance/CreditCards/query-plans.md`.
- [ ] T039 Run all `../zunera-backend/tests/Feature/CreditCards/`, relevant `../zunera-backend/tests/Feature/RecurringTransactions/`, budget/dashboard/notification tests, and `../zunera-backend/tests/Unit/CreditCards/`; verify the implemented routes against `specs/024-manage-card-purchases/contracts/purchase-statement-management-api.yaml` and fix any regression before T040.
- [ ] T040 Run the full backend suite and Pint in the live Docker container, confirming clean results for `../zunera-backend/tests/` and `../zunera-backend/app/` before any frontend task.

---

## Phase 7: User Story 1 Frontend — Move a Purchase Between Statements (P1) 🎯 MVP

**Goal**: Give card owners a clear mode, target, server preview, confirmation, refreshed statements, and move history.

**Independent Test**: From purchase history and a statement line, preview/confirm both modes and verify the same installment changes and refreshed source/destination views; dismiss or stale-preview conflict leaves the UI unchanged.

### Tests first

- [ ] T041 [P] [US1] Add failing service/store tests for options, preview, move, history, idempotency header, stale-preview error, and affected-view refresh in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js` and `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [ ] T042 [P] [US1] Add failing dialog/list/statement-line component tests for both modes, installment preselection, preview charges, disabled confirm while loading, focus return, and history in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseMoveDialog.spec.js`, `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseList.spec.js`, and `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardStatementLineItems.spec.js`.

### Implementation

- [ ] T043 [US1] Add options/preview/move/history transport and cross-view refresh actions using the verified API in `../zunera-frontend/src/services/creditCardService.js` and `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T044 [US1] Build the labelled, keyboard-accessible Full schedule/One installment move dialog with windowed future choices, server preview, stale-error feedback, confirmation, and focus restoration in `../zunera-frontend/src/components/credit-cards/CreditCardPurchaseMoveDialog.vue`.
- [ ] T045 [US1] Expose Move and move-history actions for owned purchases and preselect the clicked installment from statement lines in `../zunera-frontend/src/components/credit-cards/CreditCardPurchaseList.vue`, `../zunera-frontend/src/components/credit-cards/CreditCardStatementLineItems.vue`, and `../zunera-frontend/src/components/credit-cards/ExpandableTransactionItem.vue`.
- [ ] T046 [US1] Compose the dialog/history and refresh affected card/statement routes after success in `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue` and `../zunera-frontend/src/views/credit-cards/CreditCardStatementView.vue`.
- [ ] T047 [US1] Add pt-BR/en move labels, previews, history, success, and specific errors in `../zunera-frontend/src/i18n/messages.js`.
- [ ] T048 [US1] Add isolated fixture-backed Playwright coverage for full and individual moves, future cycles, paid surplus, stale preview, and keyboard confirmation in `../zunera-frontend/e2e/support/creditCardFixtures.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.
- [ ] T049 [US1] Run focused service/store/component tests and the isolated move journey; fix defects in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseMoveDialog.spec.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: US1 works end to end; the other purchase journeys can still be delivered separately.

---

## Phase 8: User Story 2 Frontend — Choose Statement for a Backdated Purchase (P1)

**Goal**: Show an initially empty Statement select only for a closed date-derived cycle and save the explicit allocation.

**Independent Test**: Open/closing-day dates use automatic allocation; a closed-cycle date requires a choice; date/card changes clear it; a future empty cycle saves correctly; stale options show a field error.

### Tests first

- [ ] T050 [P] [US2] Add failing service/store tests for `statement-options`, conditional `statement_closing_date`, changed-date requests, and stale-choice errors in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js` and `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [ ] T051 [P] [US2] Add a deterministic 100-case before/on/after-closing component matrix for Statement-select visibility, empty initial choice, and required selection; also test date/card reset, windowed future choices, and field-level errors in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseForm.spec.js`.

### Implementation

- [ ] T052 [US2] Add statement-option transport and conditional create payload handling using the verified API in `../zunera-frontend/src/services/creditCardService.js` and `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T053 [US2] Add labelled Statement select, loading/empty/error states, date/card-triggered clearing, and on-demand future month choices in `../zunera-frontend/src/components/credit-cards/CreditCardPurchaseForm.vue`.
- [ ] T054 [US2] Wire date/card option requests, create validation errors, and post-create refresh into `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [ ] T055 [US2] Add pt-BR/en statement-choice labels, future-cycle text, empty/loading states, and missing/stale-choice errors in `../zunera-frontend/src/i18n/messages.js`.
- [ ] T056 [US2] Add isolated Playwright cases for open and closed dates, required choice, date reset, future empty cycle, and saved allocation in `../zunera-frontend/e2e/support/creditCardFixtures.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.
- [ ] T057 [US2] Run focused form/service/store tests and the isolated create journey; fix defects in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseForm.spec.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: US2 works end to end without changing recurring purchase UI behavior.

---

## Phase 9: User Story 3 Frontend — Permanently Delete a Purchase (P2)

**Goal**: Explain permanent removal, require confirmation, and refresh financial views without hiding retained payments.

**Independent Test**: Dismiss Delete with no effect; confirm an eligible purchase and see it vanish once; verify payment/card credit remain visible; a linked-credit purchase shows the correction path.

### Tests first

- [ ] T058 [P] [US3] Add failing service/store tests for DELETE idempotency, linked-credit conflict, missing-resource replay, and affected-view refresh in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js` and `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [ ] T059 [P] [US3] Add failing confirmation component tests for permanent-removal wording, purchase name/installment count, dismissal, disabled submit, linked-credit feedback, and focus return in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseDeleteDialog.spec.js`.

### Implementation

- [ ] T060 [US3] Add idempotent delete transport, conflict handling, and card/statement refresh actions in `../zunera-frontend/src/services/creditCardService.js` and `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T061 [US3] Build accessible permanent-delete confirmation with purchase/installment details and correction guidance in `../zunera-frontend/src/components/credit-cards/CreditCardPurchaseDeleteDialog.vue`.
- [ ] T062 [US3] Expose Delete from purchase and statement views, close the dialog safely, and refresh purchases, statements, credit, and summaries after success in `../zunera-frontend/src/components/credit-cards/CreditCardPurchaseList.vue`, `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`, and `../zunera-frontend/src/views/credit-cards/CreditCardStatementView.vue`.
- [ ] T063 [US3] Add pt-BR/en permanent-delete warning, linked-credit correction path, success, and error text in `../zunera-frontend/src/i18n/messages.js`.
- [ ] T064 [US3] Add isolated Playwright delete/dismiss/linked-credit/paid-statement journeys and retained-payment assertions in `../zunera-frontend/e2e/support/creditCardFixtures.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.
- [ ] T065 [US3] Run focused dialog/service/store tests and the isolated delete journey; fix defects in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseDeleteDialog.spec.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: All three user stories work end to end.

---

## Phase 10: Polish and Cross-Cutting Verification

**Purpose**: Verify the full financial and accessible workflow without expanding the feature scope.

- [ ] T066 [P] Verify 320px/200% zoom, keyboard/focus, Light/Dark/System themes, and pt-BR/en labels for all three dialogs in `../zunera-frontend/e2e/credit-cards.spec.js` and `../zunera-frontend/src/i18n/__tests__/locale.spec.js`.
- [ ] T067 Run the SC-001/SC-006 scripted check with 20 card-feature users on fixture data (five per mode/layout cell); record anonymized completion times, 19/20 move success, 18/20 comprehension, and 30-second result discovery in `specs/024-manage-card-purchases/usability-results.md` using `specs/024-manage-card-purchases/quickstart.md`.
- [ ] T068 Run frontend Vitest/lint/build/isolated Playwright using `specs/024-manage-card-purchases/quickstart.md`, confirm the T040 backend gate is still valid, and record final results in `specs/024-manage-card-purchases/tasks.md`.

---

## Dependencies and Execution Order

1. **Setup**: T001 before application edits.
2. **Foundation**: T002–T009 before any story backend task. T002 and T003 are independent failing-test work; T004/T005 support T007; T006 supports both US1 and US2.
3. **Backend stories**: US1 T010–T021, US2 T022–T027, then US3 T028–T037. US2 can be built after foundation without the US1 move endpoint; US3 needs the shared surplus foundation and uses US1 move-history cascade when present. Each story has its own focused API test gate.
4. **Global backend gate**: T038 measures option/preview p95 before T039–T040 run all backend tests and Pint; finish the gate before T041. This feature's project rule requires the entire backend contract, authorization, validation, and tests before frontend work.
5. **Frontend stories**: US1 T041–T049, US2 T050–T057, then US3 T058–T065. Existing view/service/store files are shared; integrate in that order to avoid simultaneous edits. Each story has its own independent end-to-end checkpoint.
6. **Polish**: T066–T068 after all desired stories. T066 and T067 may proceed independently after the UI is complete; T068 follows both.

### Story dependency graph

```text
T001 Setup
  -> T002–T009 Foundation
  -> US1 backend / US2 backend / US3 backend
  -> T038–T040 Backend performance and contract gate
  -> US1 frontend -> US2 frontend -> US3 frontend
  -> T066–T068 Verification
```

US1 is the MVP. Its backend is independently testable after Foundation; its frontend waits for the global backend gate. US2 and US3 are independently testable API increments, with final UI integration using the same card views.

## Parallel Execution Examples

- **US1**: Write `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseMoveContractTest.php` and `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseMoveAccountingTest.php` in parallel; create `../zunera-backend/database/migrations/2026_10_20_000002_create_credit_card_purchase_moves_table.php` and `../zunera-backend/database/migrations/2026_10_20_000003_add_superseded_at_to_credit_card_credit_applications_table.php` in parallel; join before T016.
- **US2**: Write `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseStatementChoiceTest.php` and the recurring regression in `../zunera-backend/tests/Feature/CreditCards/RecurringCardPurchaseAccountingTest.php` in parallel; join before T024.
- **US3**: Write delete contract, delete accounting, and deleted-occurrence tests in their separate `../zunera-backend/tests/Feature/` files in parallel; join before T031.
- **Frontend**: Within each story, service/store tests and dialog/form tests may be prepared in parallel after T040. Implement shared `../zunera-frontend/src/services/creditCardService.js`, `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`, and route views sequentially by story.

## Implementation Strategy

1. Finish Setup and Foundation. Keep the existing API, account-payment behavior, and recurring generator passing.
2. Deliver all backend story slices and the global backend gate. Each slice adds a tested, owner-scoped contract; cross-story financial reconciliation remains shared.
3. Deliver US1 UI as the MVP, then US2 and US3 UI. Run each story's service/component tests and isolated Playwright journey before continuing.
4. Complete backend performance at T038–T040, then accessibility, localization, scripted usability, and final frontend checks from [quickstart.md](quickstart.md).

## Notes

- `[P]` marks work in separate files with no unfinished dependency; other tasks follow the explicit gates above.
- API paths and error codes are defined in [purchase-statement-management-api.yaml](contracts/purchase-statement-management-api.yaml).
- Use existing Laravel services/DTOs/Resources and Vue `<script setup>`/Pinia/Axios/Element Plus conventions. Keep financial calculations on the server.
- Do not edit unrelated `../zunera-frontend/package-lock.json` content.
