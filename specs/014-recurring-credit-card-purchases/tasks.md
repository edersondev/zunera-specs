# Tasks: Recurring Credit Card Purchases

**Input**: [spec.md](spec.md), [plan.md](plan.md), [research.md](research.md), [data-model.md](data-model.md), [API contract](contracts/recurring-credit-card-api.yaml), [quickstart.md](quickstart.md)

**Tests**: Required by SQR-003 and the constitution. Complete all backend contract, authorization, validation, unit/feature tests, and the backend gate before starting any frontend task. Use existing packages and project structure.

**Paths**: All paths below are relative to `zunera-specs/`. Backend work lives in `../zunera-backend/`; frontend work lives in `../zunera-frontend/`.

## Phase 1: Setup

**Purpose**: Verify coordinated branches and existing baseline. No new project or dependency setup is needed.

- [ ] T001 Verify branch `014-recurring-credit-card-purchases` in `./`, `../zunera-backend/`, and `../zunera-frontend/`; review `specs/014-recurring-credit-card-purchases/spec.md`, `docs/design/design-foundation.md`, `docs/design/app-shell.md`, `docs/design/navigation.md`, and `docs/design/components.md` before editing applications.
- [ ] T002 Run and record the existing account recurrence and card purchase test baseline from `../zunera-backend/tests/Feature/RecurringTransactions/` and `../zunera-backend/tests/Feature/CreditCards/`; inspect `../zunera-frontend/e2e/recurring-transactions.spec.js` for current behavior.

---

## Phase 2: Foundation — persistence and shared identity

**Purpose**: Build the compatible storage and model layer needed by all five stories. No story work starts until this phase passes.

- [ ] T003 Add upgrade/fresh-database tests in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardMigrationTest.php` for existing account rows, exclusive destination IDs, unique `(rule, scheduled_date)`, nullable unique purchase source, confirmation claim/choice-version persistence, and unchanged manual purchases.
- [ ] T004 Add `../zunera-backend/database/migrations/2026_09_24_000001_extend_recurring_transactions_for_credit_cards.php`: backfill/default existing rules to `financial_account`, conditionally nullable account ID, nullable card ID, card generation mode, and exclusive destination validation/constraint where supported; preserve existing keys and account data.
- [ ] T005 Add `../zunera-backend/database/migrations/2026_09_24_000002_create_recurring_card_occurrences_table.php` with owner/rule/date uniqueness, workflow state, immutable schedule/card/category/display snapshots, optional one-date overrides, durable confirmation action claim/choice version, retry/audit fields, and owner review indexes.
- [ ] T006 Add `../zunera-backend/database/migrations/2026_09_24_000003_link_credit_card_purchases_to_recurring_card_occurrences.php` with a nullable unique source reference that leaves historical/manual purchases unchanged.
- [ ] T007 Add `../zunera-backend/app/Enums/RecurringTransactions/RecurrenceDestinationType.php`, `../zunera-backend/app/Enums/RecurringTransactions/CardGenerationMode.php`, `../zunera-backend/app/Enums/RecurringTransactions/CardOccurrenceState.php`, and relationships in `../zunera-backend/app/Models/RecurringTransaction.php`, `../zunera-backend/app/Models/RecurringCardOccurrence.php`, and `../zunera-backend/app/Models/CreditCardPurchase.php`; keep original card/category identity readable after archival.
- [ ] T008 Extend owner-scoped fixtures in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringTransactionFeatureTestCase.php` and pass `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardMigrationTest.php` on fresh and upgraded schemas.

**Checkpoint**: Existing account recurrences and manual card purchases survive migration; card occurrence/source uniqueness is enforced.

---

## Phase 3: US1 backend — Schedule a Credit Card Expense (P1)

**Goal**: Create, list, filter, and view expense rules with an active owned card; preserve account rule creation and behavior.

**Independent test**: Save a monthly R$ 150 gym rule on C6 Bank •••• 3450, read card/category/mode/next date, reject foreign or inactive associations and income, and create an old-style account rule unchanged.

- [ ] T009 [P] [US1] Add create/show/list contract and legacy account compatibility tests in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardRuleContractTest.php` against `specs/014-recurring-credit-card-purchases/contracts/recurring-credit-card-api.yaml`.
- [ ] T010 [P] [US1] Add owner isolation, destination exclusivity, automatic default, expense-only, active card/category, and filter validation tests in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardRuleValidationTest.php`.
- [ ] T011 [US1] Extend create/update DTOs and conditional Form Request validation in `../zunera-backend/app/Data/RecurringTransactions/CreateRecurringTransactionData.php`, `../zunera-backend/app/Data/RecurringTransactions/UpdateRecurringTransactionData.php`, `../zunera-backend/app/Http/Requests/RecurringTransactions/StoreRecurringTransactionRequest.php`, and `../zunera-backend/app/Http/Requests/RecurringTransactions/UpdateRecurringTransactionRequest.php`; reject destination-type changes.
- [ ] T012 [US1] Extend list filters in `../zunera-backend/app/Data/RecurringTransactions/RecurringTransactionFilterData.php` and `../zunera-backend/app/Http/Requests/RecurringTransactions/ListRecurringTransactionsRequest.php` for destination/card while preserving existing filters and privacy-safe foreign-ID behavior.
- [ ] T013 [US1] Extend owner-scoped creation, same-type edit, list/count, and next-date logic in `../zunera-backend/app/Services/RecurringTransactions/RecurringTransactionService.php`; prevent rule creation from making a financial record.
- [ ] T014 [US1] Add card identity, destination type, and mode to `../zunera-backend/app/Http/Resources/RecurringTransactions/RecurringTransactionResource.php` and existing response paths in `../zunera-backend/app/Http/Controllers/Api/V1/RecurringTransactionController.php`.
- [ ] T015 [US1] Pass `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardRuleContractTest.php`, `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardRuleValidationTest.php`, `../zunera-backend/tests/Feature/RecurringTransactions/CreateAndListRecurringTransactionsTest.php`, and `../zunera-backend/tests/Feature/RecurringTransactions/AuthorizationAndIdempotencyTest.php`.

**Checkpoint**: US1 rule API and compatibility tests pass.

---

## Phase 4: US2 backend — Generate a Fixed Charge Automatically (P1)

**Goal**: Convert each eligible automatic date into at most one ordinary single-payment card purchase, including catch-up, late statement restatement, and explicit owner over-limit decisions.

**Independent test**: Process day 12 repeatedly and concurrently; verify one source-linked purchase/installment, correct statement, unchanged account balance, and an actionable over-limit item that can be approved or dismissed without duplication.

- [ ] T016 [P] [US2] Add at least 100 fixture-driven due-date cases spanning month-end/leap, catch-up, pause/resume, inclusive end date, retry, cursor, and genuinely overlapping concurrent uniqueness attempts; include a backlog where date one commits, date two fails, and date three still processes, plus an unrepresentable failure that stops only that rule in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardAutomaticGenerationTest.php`.
- [ ] T017 [P] [US2] Add at least 100 billing-boundary cases including closing-date, late closed/paid statement, available-credit, account-balance, and one-installment equivalence in `../zunera-backend/tests/Feature/CreditCards/RecurringCardPurchaseAccountingTest.php`.
- [ ] T018 [P] [US2] Add unit tests for card occurrence state transitions, per-date cursor progress, represented-date uniqueness decisions, and safe automatic retry in `../zunera-backend/tests/Unit/RecurringTransactions/RecurringCardOccurrenceStateTest.php`.
- [ ] T019 [P] [US2] Add unit tests for source-aware card purchase inputs, scheduled-date preservation, and single-payment invariants in `../zunera-backend/tests/Unit/CreditCards/RecurringCardPurchaseSourceTest.php`.
- [ ] T020 [US2] Extend `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php` with a source-aware internal mutation that reuses current card/category/credit, allocation, reconciliation, and idempotency rules; commit source link and occurrence state atomically with consistent locks.
- [ ] T021 [US2] Branch `../zunera-backend/app/Services/RecurringTransactions/RecurringOccurrenceService.php` by destination and move the card path out of its current whole-rule transaction: commit each due date, occurrence, and cursor progress independently; call source-aware purchase for automatic mode; leave over-limit dates awaiting owner approval without a purchase; persist recoverable failed identity after purchase rollback and continue later dates; stop with cursor before a date whose failure cannot be represented; retain the current account path.
- [ ] T022 [US2] Add a privacy-safe failed-automatic retry action in `../zunera-backend/app/Http/Controllers/Api/V1/RecurringTransactionController.php`, `../zunera-backend/routes/api.php`, and `../zunera-backend/app/Http/Requests/RecurringTransactions/RetryCardOccurrenceRequest.php`; retries use original date/amount and cannot silently approve over-limit.
- [ ] T023 [US2] Expose nullable recurrence source on purchase/list/statement responses via `../zunera-backend/app/Data/CreditCards/PurchaseResponseData.php` and `../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementDetailResource.php`; eager-load source through `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php` and keep manual purchase response fields intact.
- [ ] T024 [US2] Expose owner-scoped card occurrence read/list and state, source, and failure fields in `../zunera-backend/app/Http/Resources/RecurringTransactions/CardOccurrenceResource.php`, `../zunera-backend/app/Http/Controllers/Api/V1/RecurringTransactionController.php`, and `../zunera-backend/routes/api.php`; preserve the account occurrence response shape.
- [ ] T025 [US2] Add automatic awaiting-over-limit approval/dismissal with current-credit recheck, a fresh approval idempotency key, and permanent date reservation in `../zunera-backend/app/Services/RecurringTransactions/RecurringCardOccurrenceActionService.php`, `../zunera-backend/app/Http/Requests/RecurringTransactions/ConfirmCardOccurrenceRequest.php`, `../zunera-backend/app/Http/Requests/RecurringTransactions/DismissCardOccurrenceRequest.php`, `../zunera-backend/app/Http/Controllers/Api/V1/RecurringTransactionController.php`, and `../zunera-backend/routes/api.php`.
- [ ] T026 [US2] Add owner isolation, occurrence read/list, automatic approval/dismissal, stale-credit conflict, missing/invalid idempotency key, and at least 100 genuinely overlapping repeated/retry/decision attempts for one rule/date in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardAutomaticOccurrenceActionsTest.php` against `specs/014-recurring-credit-card-purchases/contracts/recurring-credit-card-api.yaml`.
- [ ] T027 [US2] Pass new automatic, accounting, unit, and automatic-action tests plus `../zunera-backend/tests/Feature/RecurringTransactions/ProcessRecurringOccurrencesTest.php` and `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseTest.php`.

**Checkpoint**: US2 automatic generation, occurrence read, approval/dismissal, and financial tests pass.

---

## Phase 5: US3 backend — Confirm an Expected Charge (P1)

**Goal**: Keep due confirmation-mode charges expected until owner confirms or dismisses; support one-date actual values and failure-safe retries.

**Independent test**: Due occurrence has no purchase or Budget expense; confirm R$ 165 on a valid actual date with eligible replacement, or dismiss it; failed recording retains choices and repeated/concurrent actions have one outcome.

- [ ] T028 [P] [US3] Add owner-scoped expected/confirm/dismiss/replay/idempotency contract tests, including failed confirmation retry, in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardConfirmationContractTest.php` for the occurrence paths in `specs/014-recurring-credit-card-purchases/contracts/recurring-credit-card-api.yaml`.
- [ ] T029 [P] [US3] Add amount/date override, future-date rejection, unavailable association replacement, current-credit recheck, confirmation over-limit recheck, and stale-confirmation tests in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardConfirmationRulesTest.php`.
- [ ] T030 [P] [US3] Add unit tests for a single active confirmation claim, choice-version matching, stale-claim recovery after checking purchase existence, retained choices, omitted-field retry defaults, explicit revision validation, and failed-to-recorded state transitions in `../zunera-backend/tests/Unit/RecurringTransactions/RecurringCardConfirmationRetryTest.php`.
- [ ] T031 [US3] Add feature tests that force a purchase failure after validated one-occurrence amount/date/card/category selection, verify no partial financial record, read back retained choices, retry with omitted fields, and record once on the chosen date; include at least 100 repeated/concurrent attempts with distinct choices, verify losers cannot overwrite the purchase's winning amount/date/card/category, and recover an interrupted claim without duplication in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardConfirmationRetryTest.php`.
- [ ] T032 [US3] Extend `../zunera-backend/app/Services/RecurringTransactions/RecurringOccurrenceService.php` to persist a snapshot-only `expected` occurrence for confirmation mode; ensure no purchase, statement line, credit effect, or account transaction exists before confirmation.
- [ ] T033 [US3] Add typed confirmation data and validation in `../zunera-backend/app/Data/RecurringTransactions/ConfirmCardOccurrenceData.php` and `../zunera-backend/app/Http/Requests/RecurringTransactions/ConfirmCardOccurrenceRequest.php`; restrict actual date to today's São Paulo business date or earlier, allow one-date overrides only in confirmation mode, and default omitted retry fields to retained choices.
- [ ] T034 [US3] Extend owner-scoped actions with expected and failed-confirmation confirm/dismiss in `../zunera-backend/app/Services/RecurringTransactions/RecurringCardOccurrenceActionService.php`; acquire one durable action claim and choice version before persisting validated owner choices, reject competing distinct actions while claimed, recheck current credit, use the claimed values for atomic purchase/recorded commit, retain choices and release claim on failure, recover stale claims only after checking source purchase existence, enforce idempotency and a fresh key after over-limit conflict, and keep already-due items actionable after pause/end.
- [ ] T035 [US3] Expose confirmation action results via `../zunera-backend/app/Http/Controllers/Api/V1/RecurringTransactionController.php` and `../zunera-backend/routes/api.php`; preserve the US2 read/list response shape.
- [ ] T036 [US3] Pass confirmation contract, rules, retry feature and unit tests, and `../zunera-backend/tests/Feature/RecurringTransactions/AuthorizationAndIdempotencyTest.php`.

**Checkpoint**: US3 confirmation, retained-choice retry, and dismissal tests pass.

---

## Phase 6: US4 backend — Manage Future Rules and Historical Purchases (P2)

**Goal**: Edit, pause, resume, end, and repair a rule without rewriting represented occurrences or historical card purchases.

**Independent test**: Record a charge, change amount/card/category/schedule/mode, pause/end, archive association, and correct/refund the individual purchase; history and statements remain valid while only eligible future dates change.

- [ ] T037 [P] [US4] Add at least 100 fixture-driven lifecycle cases spanning future-only edit, represented-date snapshot, mode edit, paused/ended due occurrence, and immutable destination; include editing amount/card/schedule after a missed due date but before the scheduler runs, pre-pause/end catch-up, and a retryable conflict when due identity cannot be stored in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardLifecycleTest.php`.
- [ ] T038 [P] [US4] Add unavailable card/category auto-pause, rule repair, one-occurrence replacement, and purchase correction/refund source-reservation tests in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardAssociationAndHistoryTest.php`; together with T037 cover at least 100 lifecycle and exception cases.
- [ ] T039 [US4] Before an owner-initiated card edit, pause, or end, catch up eligible already-due dates under the old rule through the per-date processor, then lock and recheck that no due date remains unrepresented in `../zunera-backend/app/Services/RecurringTransactions/RecurringTransactionService.php` and `../zunera-backend/app/Services/RecurringTransactions/RecurringOccurrenceService.php`; return a retryable `recurrence_due_processing_incomplete` conflict without changing the rule if representation fails, preserve earlier committed due purchases, and do not revive future dates when acting on an already-due item.
- [ ] T040 [US4] Wire unavailable-association auto-pause, eligible same-type repair/resume, and historical identity in `../zunera-backend/app/Services/RecurringTransactions/RecurringTransactionService.php` and `../zunera-backend/app/Services/CreditCards/CreditCardService.php`.
- [ ] T041 [US4] Keep purchase source immutable through correction, cancellation, and refund in `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseCorrectionService.php` and `../zunera-backend/app/Services/CreditCards/CreditCardCreditEventService.php`; never regenerate its scheduled date.
- [ ] T042 [US4] Pass `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardLifecycleTest.php`, `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardAssociationAndHistoryTest.php`, `../zunera-backend/tests/Feature/RecurringTransactions/ArchivedAssociationRecurringTransactionsTest.php`, and `../zunera-backend/tests/Feature/CreditCards/CreditCardRefundAndCreditEventTest.php`.

**Checkpoint**: US4 lifecycle, association, and historical integrity tests pass.

---

## Phase 7: US5 backend — Understand Reports Without Double Counting (P2)

**Goal**: Forecast scheduled card charges once, recognize only the linked installment as spending, and keep account, payment, transfer, refund, and card totals consistent.

**Independent test**: Compare an expected occurrence, Open-statement purchase, closed-statement installment, and later payment across Budget, Dashboard, history, account balance, obligation, and available credit.

- [ ] T043 [P] [US5] Add Open/closed/paid statement, closing-month Budget, category, correction/refund, and no-rule-double-count tests in `../zunera-backend/tests/Feature/CreditCards/RecurringCardBudgetRecognitionTest.php`.
- [ ] T044 [P] [US5] Add Dashboard upcoming/recent/summary/distribution/evolution, history, transfer exclusion, and single-source tests in `../zunera-backend/tests/Feature/FinancialDashboard/RecurringCardDashboardIntegrationTest.php`.
- [ ] T045 [P] [US5] Add unit tests for single-source Budget/Dashboard projection, upcoming suppression after recording, and statement-payment exclusion in `../zunera-backend/tests/Unit/FinancialDashboard/RecurringCardProjectionTest.php`.
- [ ] T046 [US5] Ensure source-linked single installments follow current Expected/Realized closing-month Budget rules in `../zunera-backend/app/Services/CreditCards/CreditCardBudgetProjectionService.php` and `../zunera-backend/app/Services/Budgets/BudgetCalculationService.php`; expected occurrences themselves contribute zero.
- [ ] T047 [US5] Include recognized card expenses exactly once in Dashboard totals/distribution/evolution via `../zunera-backend/app/Services/FinancialDashboard/DashboardSummaryService.php`, `../zunera-backend/app/Services/FinancialDashboard/DashboardExpenseDistributionService.php`, and `../zunera-backend/app/Services/FinancialDashboard/DashboardEvolutionService.php`, preserving transfer and payment exclusions.
- [ ] T048 [US5] Project one future/due unrecorded card expectation and one recent recorded purchase, with source-based suppression of duplicates, in `../zunera-backend/app/Services/FinancialDashboard/DashboardUpcomingActivityService.php` and `../zunera-backend/app/Services/FinancialDashboard/DashboardRecentActivityService.php`.
- [ ] T049 [US5] Preserve existing card installment, refund, and correction recognition in `../zunera-backend/app/Services/FinancialHistory/FinancialHistoryService.php` and `../zunera-backend/app/Services/CreditCards/CreditCardDashboardProjectionService.php`; include source traceability without changing account balances.
- [ ] T050 [US5] Pass new reporting feature and unit tests plus `../zunera-backend/tests/Feature/CreditCards/CreditCardBudgetIntegrationTest.php`, `../zunera-backend/tests/Feature/CreditCards/CreditCardDashboardAndHistoryIntegrationTest.php`, and `../zunera-backend/tests/Feature/FinancialDashboard/DashboardUpcomingActivityTest.php`.

**Checkpoint**: US5 Budget, Dashboard, history, and single-recognition tests pass.

---

## Phase 8: Backend contract and test gate

**Purpose**: Complete all backend API, security, compatibility, unit, feature, and query-scale checks before frontend implementation.

- [ ] T051 Validate all changed API paths, required fields, retryable `recurrence_due_processing_incomplete` and `occurrence_action_in_progress` conflicts, owner isolation, and pagination against `specs/014-recurring-credit-card-purchases/contracts/recurring-credit-card-api.yaml` using `../zunera-backend/tests/Feature/RecurringTransactions/RecurringCardApiContractTest.php`.
- [ ] T052 Run full backend suite and Pint from `../zunera-backend/composer.json` and `../zunera-backend/phpunit.xml`; fix only feature regressions under `../zunera-backend/app/` and `../zunera-backend/tests/`.
- [ ] T053 Measure 1,000-rule list/detail and occurrence-query behavior in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringTransactionScaleTest.php`, checking query counts and avoiding per-rule lookups; browser usability p95 is measured after frontend delivery.

**Checkpoint**: Every backend story, full suite, documented contract, and backend query-scale check pass. Only then begin Phase 9.

---

## Phase 9: US1 frontend — Schedule a Credit Card Expense (P1)

**Goal**: Use the verified rule API in the existing recurrence form, list, filters, and detail.

**Independent test**: Create and inspect account and card rules, with mode/card identity and no purchase on save.

- [ ] T054 [US1] Add form/list/detail/service/store tests for immutable destination, active card selection, account regression, and card identity in `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringTransactionFormDialog.spec.js`, `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringTransactionList.spec.js`, `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringTransactionDetailDrawer.spec.js`, `../zunera-frontend/src/services/__tests__/recurringTransactionService.spec.js`, and `../zunera-frontend/src/stores/recurring-transactions/__tests__/recurringTransactionStore.spec.js`.
- [ ] T055 [US1] Add destination/account/card selector, automatic-default and confirmation choice, expense-only validation, and recurrence-versus-installments guidance in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionFormDialog.vue`.
- [ ] T056 [US1] Add card/destination filters and textual card identity in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionFilterBar.vue`, `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionList.vue`, and `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionDetailDrawer.vue` using `../zunera-frontend/src/utils/credit-cards/creditCardFormatters.js`.
- [ ] T057 [US1] Wire owned card options and rule fields through `../zunera-frontend/src/views/recurring-transactions/RecurringTransactionsListView.vue`, `../zunera-frontend/src/stores/recurring-transactions/recurringTransactionStore.js`, and `../zunera-frontend/src/services/recurringTransactionService.js`; replace the unsupported-card message and update `../zunera-frontend/src/i18n/messages.js`.
- [ ] T058 [US1] Cover account/card rule creation and readback in `../zunera-frontend/e2e/recurring-transactions.spec.js`; update the obsolete exclusion assertion in `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: US1 configuration journey passes against the verified backend.

---

## Phase 10: US2 frontend — Generate a Fixed Charge Automatically (P1)

**Goal**: Show the automatic purchase source and make over-limit decisions actionable in the existing recurrence detail.

**Independent test**: View recorded/failed/awaiting-over-limit occurrences, approve or dismiss an over-limit item, and navigate to its single purchase and statement.

- [ ] T059 [US2] Add service/store tests for occurrence read/list, failed automatic retry, awaiting-over-limit approval and dismissal, typed stale-credit conflict, and fresh idempotency keys in `../zunera-frontend/src/services/__tests__/recurringTransactionService.spec.js` and `../zunera-frontend/src/stores/recurring-transactions/__tests__/recurringTransactionStore.spec.js`.
- [ ] T060 [US2] Add recorded-origin, source-link, and automatic approval/dismissal tests in `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringTransactionDetailDrawer.spec.js` and `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseList.spec.js`.
- [ ] T061 [US2] Add UI tests for automatic awaiting-over-limit review, explicit approval, dismissal, and retryable failure without duplicate actions in `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringCardOccurrenceDialog.spec.js`.
- [ ] T062 [US2] Wire occurrence read/list, automatic retry, approval, dismissal, and fresh idempotency keys through `../zunera-frontend/src/services/recurringTransactionService.js` and `../zunera-frontend/src/stores/recurring-transactions/recurringTransactionStore.js`.
- [ ] T063 [US2] Show recorded occurrence, original scheduled date, purchase link, awaiting-over-limit approval/dismissal, and retryable failure without an ordinary account transaction link in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionDetailDrawer.vue` and new `../zunera-frontend/src/components/recurring-transactions/RecurringCardOccurrenceDialog.vue`.
- [ ] T064 [US2] Show recurrence source and navigate back to its rule in `../zunera-frontend/src/components/credit-cards/CreditCardPurchaseList.vue`, `../zunera-frontend/src/components/credit-cards/CreditCardStatementLineItems.vue`, and `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [ ] T065 [US2] Cover automatic purchase origin, statement placement, source navigation, over-limit approval/dismissal, and no duplicate obligation in `../zunera-frontend/e2e/recurring-transactions.spec.js` and `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: US2 automatic and over-limit journeys pass against the verified backend.

---

## Phase 11: US3 frontend — Confirm an Expected Charge (P1)

**Goal**: Let owners confirm or dismiss a due charge with one-occurrence values and inspect retained choices after failure.

**Independent test**: Confirm with an actual amount/date or dismiss; after simulated failure, retry with retained values and record one purchase.

- [ ] T066 [US3] Add confirmation-mode confirm/dismiss/failed retry, typed 409 over-limit, and retryable `occurrence_action_in_progress` service/store tests in `../zunera-frontend/src/services/__tests__/recurringTransactionService.spec.js` and `../zunera-frontend/src/stores/recurring-transactions/__tests__/recurringTransactionStore.spec.js`.
- [ ] T067 [US3] Extend occurrence actions with confirmation and retained-choice retry-safe store state in `../zunera-frontend/src/services/recurringTransactionService.js` and `../zunera-frontend/src/stores/recurring-transactions/recurringTransactionStore.js`.
- [ ] T068 [US3] Add confirmation UI tests for future-date rejection, confirmation over-limit warning, retryable active-attempt conflict, retained failed choices, explicit retry revision, one-date overrides, and no issuer-verification claim in `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringCardOccurrenceDialog.spec.js`.
- [ ] T069 [US3] Extend expected/failed/dismissed/recorded review and accessible confirmation of one-date amount, date, card, and category with retained failed-attempt choices and retryable active-attempt feedback in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionDetailDrawer.vue` and `../zunera-frontend/src/components/recurring-transactions/RecurringCardOccurrenceDialog.vue`.
- [ ] T070 [US3] Cover due expected item, confirm and dismiss, failed-choice retry, confirmation over-limit handling, and no duplicate purchase in `../zunera-frontend/e2e/recurring-transactions.spec.js`.

**Checkpoint**: US3 variable confirmation, failure retry, and dismissal journeys pass.

---

## Phase 12: US4 frontend — Manage Future Rules and Historical Purchases (P2)

**Goal**: Show future-only edits and historical source identity through lifecycle changes.

**Independent test**: Edit and pause a rule after a purchase; verify the past charge and due occurrence remain readable and correct.

- [ ] T071 [US4] Add immutable destination, future-only edit, paused/ended due action, retryable incomplete-due-processing feedback, and unavailable-association tests in `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringTransactionFormDialog.spec.js`, `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringTransactionLifecycleDialog.spec.js`, and `../zunera-frontend/src/components/recurring-transactions/__tests__/RecurringTransactionDetailDrawer.spec.js`.
- [ ] T072 [US4] Show original versus effective one-date association, actionable repair/pause/end behavior, and retryable incomplete-due-processing feedback in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionFormDialog.vue`, `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionLifecycleDialog.vue`, `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionDetailDrawer.vue`, and `../zunera-frontend/src/i18n/messages.js`.
- [ ] T073 [US4] Cover rule edit after a missed due date, pause/resume/end, unavailable association repair, retryable incomplete-due-processing conflict, and unchanged historical purchase in `../zunera-frontend/e2e/recurring-transactions.spec.js`.

**Checkpoint**: US4 lifecycle and history journeys pass.

---

## Phase 13: US5 frontend — Understand Reports Without Double Counting (P2)

**Goal**: Display forecast and recorded card activity with one recognized expense source.

**Independent test**: Compare expected, Open, closed, and paid views without duplicated Budget or Dashboard spending.

- [ ] T074 [US5] Add expected-versus-recorded and recurrence-source display tests in `../zunera-frontend/src/components/dashboard/__tests__/UpcomingActivityCard.spec.js`, `../zunera-frontend/src/components/dashboard/__tests__/RecentActivityCard.spec.js`, and budget regression tests in `../zunera-frontend/src/views/budgets/__tests__/BudgetsView.spec.js`.
- [ ] T075 [US5] Label card expectations, recorded purchases, and source links without color-only meaning in `../zunera-frontend/src/components/dashboard/UpcomingActivityCard.vue`, `../zunera-frontend/src/components/dashboard/RecentActivityCard.vue`, and `../zunera-frontend/src/utils/dashboard/dashboardFormatters.js`; leave Budget category display tied to existing purchase recognition.
- [ ] T076 [US5] Cover expected → Open → closed → paid across Dashboard/Budget/card views without duplicate spending in `../zunera-frontend/e2e/recurring-transactions.spec.js`.

**Checkpoint**: US5 reporting journeys pass.

---

## Phase 14: Final frontend and outcome verification

- [ ] T077 Run frontend unit tests, lint, build, and focused Playwright journeys with isolated scenarios and accessible selectors from `../zunera-frontend/package.json`, `../zunera-frontend/e2e/recurring-transactions.spec.js`, and `../zunera-frontend/e2e/credit-cards.spec.js`; fix feature regressions.
- [ ] T078 Verify responsive, 200% zoom, keyboard/focus, assistive labels, and Light/Dark/System states against `docs/design/design-foundation.md`, `docs/design/app-shell.md`, `docs/design/navigation.md`, and `docs/design/components.md` in `../zunera-frontend/src/components/recurring-transactions/RecurringCardOccurrenceDialog.vue` and `../zunera-frontend/src/components/dashboard/UpcomingActivityCard.vue`.
- [ ] T079 [P] Measure browser list and detail usability with 1,000 representative owned rules, at least 20 timings per supported desktop and narrow mobile viewport after warmup, and p95 at most 2 seconds in `../zunera-frontend/e2e/recurring-transactions-performance.spec.js`; record device/network conditions and verify primary controls are usable.
- [ ] T080 Execute the acceptance walkthrough in `specs/014-recurring-credit-card-purchases/quickstart.md`, including the at-least-20-owner timed protocol for SC-001/SC-006, at-least-100-case protocols for SC-002–SC-005, and finance edge cases; record any gaps in `specs/014-recurring-credit-card-purchases/quickstart.md`.

---

## Dependencies and execution order

```text
Setup → Foundation
  → US1 backend → US2 backend → US3 backend → US4 backend → US5 backend
  → Backend contract and test gate
  → US1 frontend → US2 frontend → US3 frontend → US4 frontend → US5 frontend
  → Final frontend and outcome verification
```

- No frontend task starts until the full backend gate passes, including US1–US5 authorization, validation, contract, unit, and feature coverage. This is the constitution-required delivery order.
- US2 requires the US1 card rule and Foundation source identity; its automatic path includes occurrence read/list and owner over-limit approval/dismissal.
- US3 extends US2 source-aware purchase and action services for expected and failed-confirmation items.
- US4 uses prior rule/occurrence/purchase paths. US5 consumes their recognized sources.
- Tasks touching the same service or UI file run sequentially even when tests in separate files are marked [P].

### Requirement coverage

| Story | Main functional requirements |
|---|---|
| US1 | FR-001–FR-005, FR-020, FR-025 |
| US2 | FR-008–FR-009, FR-011–FR-016, FR-027 |
| US3 | FR-006–FR-007, FR-010–FR-014, FR-022–FR-023, FR-026–FR-027 |
| US4 | FR-002, FR-021–FR-024, FR-026 |
| US5 | FR-007, FR-016–FR-020 |

SQR-001–SQR-005 run through story tests and the backend/frontend gates. SC-001–SC-007 have explicit measurement tasks and the protocol in `quickstart.md`.

## Implementation strategy

**Backend milestone:** Complete Foundation, US1–US5 backend, then the backend gate. A passing gate fixes the API and financial contract before any frontend implementation.

**Frontend milestone:** Deliver US1–US5 interface journeys against that contract, then run browser performance, accessibility, regression, and owner-outcome checks. No issuer connection, automatic charge, statement payment, or new installment system enters scope.
