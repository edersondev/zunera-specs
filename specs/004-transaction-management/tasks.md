# Tasks: Transaction Management

**Input**: Design documents from `specs/004-transaction-management/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md),
[research.md](research.md), [data-model.md](data-model.md),
[transactions-api.yaml](contracts/transactions-api.yaml), and
[quickstart.md](quickstart.md)

**Tests**: Tests are required by the specification's SQR-004 and the
constitution. Add Laravel feature/unit tests, frontend service/store/view
tests, contract linting, and isolated Playwright journeys with accessible
selectors.

**Organization**: Backend tasks for each story complete before that story's
frontend tasks. Stories are independently testable at their checkpoints, except
the single cross-story balance test called out in T055.

**Branch coordination**: Before work, confirm `004-transaction-management` in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Different files and no incomplete-task dependency.
- **[Story]**: User story traceability label.
- Every task includes its exact target path.
- Task IDs follow execution order and are never reused.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm shared feature scope and prepare focused domain locations.

- [ ] T001 Verify branch `004-transaction-management` in `.`,
  `../zunera-backend`, and `../zunera-frontend` before editing application
  files.
- [ ] T002 [P] Create transaction domain folders in
  `../zunera-backend/app/{Data,Enums,Exceptions,Http/Requests,Http/Resources,Services}/Transactions/`
  and `../zunera-backend/tests/{Feature,Unit}/Transactions/`.
- [ ] T003 [P] Create transaction feature folders in
  `../zunera-frontend/src/{components,stores,utils,views}/transactions/`
  without adding dependencies.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish the shared transaction domain before any user story.
All user-story work waits for this phase.

- [ ] T004 [P] Create `TransactionType` (`income`, `expense`) and
  `TransactionStatus` (`pending`, `effective`) enums in
  `../zunera-backend/app/Enums/Transactions/`.
- [ ] T005 [P] Create positive-amount bounds helper with
  `MIN_CENTAVOS = 1` and `MAX_CENTAVOS = 99_999_999_999` in
  `../zunera-backend/app/Services/Transactions/TransactionMoney.php`.
- [ ] T006 [P] Create date parsing and bounds helper for ISO `YYYY-MM-DD` and
  Brazilian `DD/MM/YYYY` input, the 1900-01-01..2100-12-31 range, and the
  future-date check in
  `../zunera-backend/app/Services/Transactions/TransactionDateRange.php`.
- [ ] T007 [P] Create lowercased, accent-stripped description-plus-notes
  normalization used by the derived `search_text` column in
  `../zunera-backend/app/Services/Transactions/TransactionTextNormalizer.php`.
- [ ] T008 [P] Create typed transaction state exception with
  `transaction_already_removed`, `transaction_already_active`,
  `transaction_edit_requires_restore`, and `transaction_state_not_allowed` codes
  in `../zunera-backend/app/Exceptions/Transactions/TransactionStateException.php`.
- [ ] T009 Create the transactions and transaction-mutation-request tables:
  transaction ownership, associations, type, status, monetary and search fields,
  removal state, history indexes, plus user-scoped idempotency key, request
  fingerprint, target transaction, and replayable response fields with a unique
  owner/key constraint in
  `../zunera-backend/database/migrations/*_create_transactions_table.php` and
  `../zunera-backend/database/migrations/*_create_transaction_mutation_requests_table.php`.
- [ ] T010 Create the guarded `Transaction` model with casts, `user`,
  `financialAccount`, and `category` relations, `search_text` rebuild hook,
  and active/removed/effective/counting scopes in
  `../zunera-backend/app/Models/Transaction.php`.
- [ ] T011 [P] Create `TransactionFactory` with income, expense, pending,
  effective, removed, future-dated, and archived-account or archived-category
  states in `../zunera-backend/database/factories/TransactionFactory.php`.
- [ ] T012 [P] Create `CreateTransactionData`, `UpdateTransactionData`, and
  `TransactionFilterData` in
  `../zunera-backend/app/Data/Transactions/`.
- [ ] T013 [P] Create `ListTransactionsRequest`, `StoreTransactionRequest`,
  `UpdateTransactionRequest`, and `LifecycleTransactionRequest` skeletons in
  `../zunera-backend/app/Http/Requests/Transactions/`, including required
  `Idempotency-Key` header validation for mutations.
- [ ] T014 Implement `TransactionBalanceReconciler` that applies the previous
  and resulting effect to one or two locked accounts exactly once, and
  `TransactionIdempotencyService` that acquires/replays owner-scoped mutation
  requests inside the same database transaction, in
  `../zunera-backend/app/Services/Transactions/TransactionBalanceReconciler.php`
  and `../zunera-backend/app/Services/Transactions/TransactionIdempotencyService.php`.
- [ ] T015 Create `TransactionResource` exposing owner-safe fields plus embedded
  account and category summaries with lifecycle status in
  `../zunera-backend/app/Http/Resources/Transactions/TransactionResource.php`.
- [ ] T016 Register the six transaction routes inside the existing
  `auth:sanctum` and `session.lifetime` group in
  `../zunera-backend/routes/api.php`.
- [ ] T017 Create the thin `TransactionController` with `index`, `store`,
  `show`, `update`, `remove`, and `restore` actions in
  `../zunera-backend/app/Http/Controllers/Api/V1/TransactionController.php`.
- [ ] T018 [P] Lint the contract with
  `npx @redocly/cli lint specs/004-transaction-management/contracts/transactions-api.yaml`
  and record the result in
  `specs/004-transaction-management/quickstart.md` planning evidence.

**Checkpoint**: Schema, enums, resource, routes, and balance helper exist; user
stories can begin.

---

## Phase 3: User Story 1 - Record Income and Expense Transactions (Priority: P1) 🎯 MVP

**Goal**: An authenticated user records income and expense transactions with
description, amount, date, account, category, and optional note, and sees them
in a basic newest-first history.

**Independent Test**: Record an income and an expense transaction through the
API and the UI, confirm each appears with description, amount, type, date,
account, and category, and confirm invalid input is rejected with field-level
feedback while unauthenticated access is denied.

### Tests for User Story 1

- [ ] T019 [P] [US1] Feature test for income and expense creation, note
  persistence, field-level validation failures, and idempotent replay of a
  create request (including concurrent/retried requests with one balance effect) in
  `../zunera-backend/tests/Feature/Transactions/RecordTransactionsTest.php`.
- [ ] T020 [P] [US1] Unit test for amount bounds, future-dated pending default,
  and rejection of explicit `effective` on a future date in
  `../zunera-backend/tests/Unit/Transactions/TransactionRulesTest.php`.

### Implementation for User Story 1

#### Backend (complete first)

- [ ] T021 [US1] Implement `TransactionService::create` with owner-scoped
  account and category resolution, active-association checks, type/category
  match, future-date pending rule, rejection of explicit `effective` on future
  dates, `has_financial_movements` and `has_financial_transactions` flags,
  persisted idempotency replay/conflict behavior, and balance reconciliation in
  `../zunera-backend/app/Services/Transactions/TransactionService.php`.
- [ ] T022 [US1] Implement `StoreTransactionRequest` validation rules, DTO
  conversion, `Idempotency-Key` validation, and privacy-safe 404 mapping for foreign accounts or categories
  in `../zunera-backend/app/Http/Requests/Transactions/StoreTransactionRequest.php`.
- [ ] T023 [US1] Wire `TransactionController::store` with the created-resource
  response in
  `../zunera-backend/app/Http/Controllers/Api/V1/TransactionController.php`.
- [ ] T024 [US1] Run `php artisan test --compact tests/Feature/Transactions
  tests/Unit/Transactions` and `vendor/bin/pint --dirty --format agent` in
  `../zunera-backend`.

#### Frontend (after Backend)

- [ ] T025 [P] [US1] Implement `createTransaction`, `listTransactions`, and
  `getTransaction` calls, generating one idempotency key per logical mutation
  and retaining it only for an exact retry, in
  `../zunera-frontend/src/services/transactionService.js`.
- [ ] T026 [US1] Implement setup-style `transactionStore` with list state,
  active transaction, and create action in
  `../zunera-frontend/src/stores/transactions/transactionStore.js`.
- [ ] T027 [US1] Build the record form dialog with Element Plus controls,
  Brazilian amount input, date picker, account and category selection filtered
  by type, optional note, field-level feedback, and duplicate-submit protection
  that disables the submit action while a save is in flight in
  `../zunera-frontend/src/components/transactions/TransactionFormDialog.vue`.
- [ ] T028 [US1] Build the basic newest-first history list showing description,
  amount, type, date, account, category, and status with income/expense
  distinguished by sign or icon as well as color, and with Brazilian amount
  formatting from
  `../zunera-frontend/src/utils/transactions/transactionFormatters.js` applied
  in `../zunera-frontend/src/views/transactions/TransactionsListView.vue`.
- [ ] T029 [US1] Add `/app/transactions` route, navigation entry, and i18n keys
  in `../zunera-frontend/src/router/index.js`,
  `../zunera-frontend/src/composables/`, and
  `../zunera-frontend/src/i18n/`.
- [ ] T030 [P] [US1] Add service, store, and formatter unit tests covering
  amount and date display plus the in-flight submit guard in
  `../zunera-frontend/src/services/__tests__/transactionService.spec.js`,
  `../zunera-frontend/src/stores/transactions/__tests__/transactionStore.spec.js`,
  and `../zunera-frontend/src/utils/transactions/__tests__/`.
- [ ] T031 [US1] Add Playwright journey that records an income and an expense
  transaction and verifies them in history in
  `../zunera-frontend/e2e/transactions.spec.js`.
- [ ] T032 [US1] Run `npm run test:unit -- --run`, `npm run build`, and
  `CI=1 npm run test:e2e -- e2e/transactions.spec.js` in
  `../zunera-frontend`.

**Checkpoint**: User Story 1 is fully functional and independently testable.

---

## Phase 4: User Story 2 - Keep Account Balances Accurate (Priority: P1)

**Goal**: Every transaction change leaves each affected account balance exactly
equal to its initial balance plus its effective, non-removed movements.

**Independent Test**: Starting from a known balance, create, flip status, change
type, move accounts, remove, and restore a transaction, asserting the expected
balance after each step.

### Tests for User Story 2

- [ ] T033 [P] [US2] Unit test the reconciliation matrix (income, expense,
  pending, effective, removal, restore, type change, account move, no-op
  updates) in
  `../zunera-backend/tests/Unit/Transactions/TransactionBalanceReconcilerTest.php`.
- [ ] T034 [P] [US2] Feature test balance effects for creation and status
  transitions, including the account summary endpoint, in
  `../zunera-backend/tests/Feature/Transactions/BalanceConsistencyTest.php`.
### Implementation for User Story 2

#### Backend (complete first)

- [ ] T036 [US2] Integrate reconciliation for create and status changes, and
  establish the locked-account, exactly-once interface consumed by the later
  update, account-move, removal, and restore paths in
  `../zunera-backend/app/Services/Transactions/TransactionService.php`.
- [ ] T037 [US2] Verify account list, show, and summary responses reflect
  reconciled balances without changing the financial-account contract in
  `../zunera-backend/app/Services/FinancialAccounts/FinancialAccountService.php`.

#### Frontend (after Backend)

- [ ] T038 [US2] Refresh the financial accounts store after every transaction
  mutation and show balance-impact feedback in
  `../zunera-frontend/src/stores/financial-accounts/` and
  `../zunera-frontend/src/components/transactions/`.
- [ ] T039 [P] [US2] Add unit tests proving the accounts store refreshes after
  transaction mutations in
  `../zunera-frontend/src/stores/transactions/__tests__/`.
- [ ] T040 [US2] Extend `../zunera-frontend/e2e/transactions.spec.js` to assert
  account balances after create, edit, and removal.
- [ ] T041 [US2] Run focused backend and frontend tests for balance behavior.

**Checkpoint**: Balances are provably consistent for every implemented
mutation path.

---

## Phase 5: User Story 3 - View and Inspect Transaction History (Priority: P2)

**Goal**: Users review their history newest first with a matching count,
progressive loading, a clear empty state, and a details view.

**Independent Test**: With several transactions, open the history, confirm
ordering and complete entry information, load further batches, open one
transaction's details, and confirm another user's transactions are never shown.

### Tests for User Story 3

- [ ] T042 [P] [US3] Feature test for history ordering, entry fields, pagination
  metadata, per-page cap of 50, and owner isolation in
  `../zunera-backend/tests/Feature/Transactions/ListTransactionHistoryTest.php`.
- [ ] T043 [P] [US3] Feature test for transaction details, privacy-safe 404 for
  another user's transaction, and unauthenticated denial in
  `../zunera-backend/tests/Feature/Transactions/ViewTransactionDetailsTest.php`.
- [ ] T044 [P] [US3] Feature test for history scale: seed 5,000 transactions for
  one user and assert the first page returns exactly the 50 newest entries,
  that `meta.total` is correct, and that every transaction stays reachable
  through pagination, search, and filters within the SC-010 budget in
  `../zunera-backend/tests/Feature/Transactions/TransactionHistoryScaleTest.php`.

### Implementation for User Story 3

#### Backend (complete first)

- [ ] T045 [US3] Implement paginated `TransactionService::list`, `findOwned`,
  and the matching-count metadata in
  `../zunera-backend/app/Services/Transactions/TransactionService.php`.
- [ ] T046 [US3] Implement `ListTransactionsRequest` pagination validation with
  a default and maximum page size of 50 in
  `../zunera-backend/app/Http/Requests/Transactions/ListTransactionsRequest.php`.
- [ ] T047 [US3] Wire `TransactionController::index` and `show` in
  `../zunera-backend/app/Http/Controllers/Api/V1/TransactionController.php`
  and run the focused history and scale tests.

#### Frontend (after Backend)

- [ ] T048 [US3] Add progressive loading, matching-count display, loading,
  error, and empty states to
  `../zunera-frontend/src/views/transactions/TransactionsListView.vue`.
- [ ] T049 [US3] Build the transaction details drawer with all recorded fields,
  status, note, and archived association labels in
  `../zunera-frontend/src/components/transactions/TransactionDetailDrawer.vue`.
- [ ] T050 [P] [US3] Add view and component unit tests for list states, ordering
  display, and detail rendering in
  `../zunera-frontend/src/views/__tests__/` and
  `../zunera-frontend/src/components/transactions/__tests__/`.
- [ ] T051 [US3] Extend `../zunera-frontend/e2e/transactions.spec.js` with
  history order, detail open, empty state, and cross-user isolation journeys.

**Checkpoint**: History and details work independently of editing and
filtering.

---

## Phase 6: User Story 4 - Correct and Manage Existing Transactions (Priority: P2)

**Goal**: Users correct every field of an owned transaction, keep archived
associations, remove transactions, and restore them from a removed view.

**Independent Test**: Edit description, note, amount, type, category, account,
date, and status; confirm archived associations stay valid and editable; remove
a transaction and confirm it leaves balances and normal history but returns by
restore.

### Tests for User Story 4

- [ ] T052 [P] [US4] Feature test for archived associations: transactions stay
  visible, editable, and correctly balanced after their account or category is
  archived, and archiving an account or category that still has pending
  transactions is not blocked and changes no state or balance, in
  `../zunera-backend/tests/Feature/Transactions/ArchivedAssociationTransactionsTest.php`.
- [ ] T053 [P] [US4] Feature test for each editable field, type/category
  mismatch, foreign-resource 404, removed-edit conflict, future-date edit
  semantics, and the typed `effective_future_date` response notice in
  `../zunera-backend/tests/Feature/Transactions/UpdateTransactionsTest.php`.
- [ ] T054 [P] [US4] Feature test for removal, removed view, restore, repeated
  lifecycle conflicts, idempotent retry/reused-key conflicts, and state rules on restore in
  `../zunera-backend/tests/Feature/Transactions/RemoveAndRestoreTransactionsTest.php`.

- [ ] T055 [US2] Feature test balance effects for type change, account move,
  removal, and restore, reusing the update and lifecycle endpoints delivered in
  User Story 4 (cross-story verification after T057, T058, and T059).

### Implementation for User Story 4

#### Backend (complete first)

- [ ] T056 [US4] Implement `UpdateTransactionRequest` rules including the
  keep-archived-association allowance, rejection of newly selected archived
  resources, `Idempotency-Key` validation, and DTO conversion in
  `../zunera-backend/app/Http/Requests/Transactions/UpdateTransactionRequest.php`.
- [ ] T057 [US4] Implement `TransactionService::update` with change detection,
  both-account reconciliation, archived-association rules, date/status edit
  rules, removed-state rejection, idempotency replay/conflict behavior, and
  `effective_future_date` response metadata in
  `../zunera-backend/app/Services/Transactions/TransactionService.php`.
- [ ] T058 [US4] Implement `TransactionService::remove` and `restore` with state
  conflicts, idempotency replay/conflict behavior, the removed view scope, and
  exactly-once reconciliation in
  `../zunera-backend/app/Services/Transactions/TransactionService.php`.
- [ ] T059 [US4] Wire `TransactionController::update`, `remove`, and `restore`
  with typed conflict responses in
  `../zunera-backend/app/Http/Controllers/Api/V1/TransactionController.php`,
  then run the focused update and lifecycle tests.

#### Frontend (after Backend)

- [ ] T060 [US4] Extend
  `../zunera-frontend/src/components/transactions/TransactionFormDialog.vue`
  for editing, archived read-only associations, the future-date notice, and the
  typed response notice plus same duplicate-submit protection used when recording.
- [ ] T061 [US4] Add `updateTransaction`, `removeTransaction`, and
  `restoreTransaction` with per-action idempotency-key generation/retry to
  `../zunera-frontend/src/services/transactionService.js` and matching store
  actions in
  `../zunera-frontend/src/stores/transactions/transactionStore.js`.
- [ ] T062 [US4] Build the removed-transactions view with Restore actions,
  route, navigation entry, success and rejection feedback, and i18n keys in
  `../zunera-frontend/src/views/transactions/RemovedTransactionsView.vue`,
  `../zunera-frontend/src/router/index.js`, and
  `../zunera-frontend/src/i18n/`.
- [ ] T063 [P] [US4] Add unit tests for update, remove, restore, typed date notice,
  idempotency-key reuse, and removed-view behavior in
  `../zunera-frontend/src/components/transactions/__tests__/` and
  `../zunera-frontend/src/views/__tests__/`.
- [ ] T064 [US4] Extend `../zunera-frontend/e2e/transactions.spec.js` with edit,
  remove, restore, removed-view, and archived-association journeys.

**Checkpoint**: Correcting and removing transactions works with balances intact.

---

## Phase 7: User Story 5 - Find Transactions by Filter and Search (Priority: P3)

**Goal**: Users narrow history by date range, type, account, category, and
status, combine filters, and search description and notes.

**Independent Test**: With a mixed history, apply each filter, combine filters,
search accented and unaccented text, clear the criteria, and confirm only
matching transactions are listed with an accurate matching count.

### Tests for User Story 5

- [ ] T065 [P] [US5] Feature test for each filter, ISO and Brazilian date input,
  inclusive date ranges, archived account and category filters, foreign account
  and category privacy-safe 404s, status, combined filters, and the
  removed view in
  `../zunera-backend/tests/Feature/Transactions/FilterTransactionsTest.php`.
- [ ] T066 [P] [US5] Feature test for description and note search, case and
  accent insensitivity, search combined with filters, and no-match results in
  `../zunera-backend/tests/Feature/Transactions/SearchTransactionsTest.php`.

### Implementation for User Story 5

#### Backend (complete first)

- [ ] T067 [US5] Implement filter and search parsing into `TransactionFilterData`
  in
  `../zunera-backend/app/Http/Requests/Transactions/ListTransactionsRequest.php`
  and `../zunera-backend/app/Data/Transactions/TransactionFilterData.php`.
- [ ] T068 [US5] Implement filter, normalized `search_text`, and removed-view
  query building with owner-scoped account/category filter resolution that maps
  foreign or unavailable IDs to 404, and stable ordering in
  `../zunera-backend/app/Services/Transactions/TransactionService.php`, then
  run the focused filter and search tests.

#### Frontend (after Backend)

- [ ] T069 [US5] Build the filter bar with date range, type, account, category,
  status, and text search controls in
  `../zunera-frontend/src/components/transactions/TransactionFilterBar.vue`.
- [ ] T070 [US5] Keep filters in transaction store state and in the route query
  string, show active criteria with i18n labels, provide a clear action, and
  render the no-match empty state in
  `../zunera-frontend/src/stores/transactions/transactionStore.js` and
  `../zunera-frontend/src/views/transactions/TransactionsListView.vue`.
- [ ] T071 [P] [US5] Add unit tests for filter state, query-string sync, and
  clearing filters in
  `../zunera-frontend/src/stores/transactions/__tests__/` and
  `../zunera-frontend/src/components/transactions/__tests__/`.
- [ ] T072 [US5] Extend `../zunera-frontend/e2e/transactions.spec.js` with
  combined filter, search, and clear-criteria journeys.

**Checkpoint**: All five stories work independently and together.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Verification and consistency work spanning all stories.

- [ ] T073 [P] Lint the contract with
  `npx @redocly/cli lint specs/004-transaction-management/contracts/transactions-api.yaml`
  and resolve any response-shape drift.
- [ ] T074 [P] Update
  `specs/004-transaction-management/{research.md,data-model.md,quickstart.md}`
  if implementation reveals a decision or field change.
- [ ] T075 [P] Run `php artisan test --compact` and
  `vendor/bin/pint --format agent` in `../zunera-backend`.
- [ ] T076 [P] Run `npm run test:unit -- --run` and `npm run build` in
  `../zunera-frontend`.
- [ ] T077 Run `CI=1 npm run test:e2e -- e2e/transactions.spec.js` in
  `../zunera-frontend` and confirm scenario isolation.
- [ ] T078 Verify Light, Dark, and System themes, narrow supported viewports,
  200% zoom, keyboard-only operation, dialog and drawer focus restoration, and
  income/expense distinction without color in
  `../zunera-frontend/src/views/transactions/`.
- [ ] T079 Run the manual acceptance smoke test in
  `specs/004-transaction-management/quickstart.md`, then mark the
  implementation-verification checkbox with evidence.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies.
- **Foundational (Phase 2)**: Depends on Setup; blocks every user story.
- **User Stories (Phases 3-7)**: Depend on Phase 2.
- **Polish (Phase 8)**: Depends on the stories you intend to ship.

### User Story Dependencies

- **US1 (P1)**: After Phase 2; no story dependencies. Includes the minimal
  list needed to see a recorded transaction.
- **US2 (P1)**: After Phase 2 for unit and create-path balance coverage; only
  T055 needs the US4 update, remove, and restore endpoints (T057-T059), so it is
  the single cross-story task in this plan.
- **US3 (P2)**: After Phase 2; replaces US1's minimal list presentation with
  full history, details, progressive loading, and scale verification.
- **US4 (P2)**: After Phase 2; requires the US1 create path to edit.
- **US5 (P3)**: After Phase 2; requires the US3 list contract.

### Within Each Story

- Backend contract, authorization, validation, and tests before frontend work.
- Models and DTOs before services; services before controllers; controller
  wiring before frontend consumption.
- Tests are written and failing before implementation where practical, and
  always executed before the story is called complete.

### Parallel Opportunities

- T002 and T003 are independent folder setup.
- T004-T008, T011-T013, and T018 touch different files and can run in parallel.
- Test tasks T019/T020 (US1), T033/T034 (US2), T042/T043/T044 (US3),
  T052/T053/T054 (US4), and T065/T066 (US5) can each run in parallel.
- Frontend unit-test tasks marked [P] run alongside component work on other
  files.

---

## Parallel Example: User Story 1

```bash
# Backend tests for User Story 1 together:
Task: "Feature test for income and expense creation in ../zunera-backend/tests/Feature/Transactions/RecordTransactionsTest.php"
Task: "Unit test for amount bounds and status defaults in ../zunera-backend/tests/Unit/Transactions/TransactionRulesTest.php"

# Frontend groundwork together:
Task: "Implement transaction service calls in ../zunera-frontend/src/services/transactionService.js"
Task: "Add service, store, and formatter unit tests in ../zunera-frontend/src/services/__tests__/"
```

---

## Implementation Strategy

### MVP First (User Story 1)

1. Complete Phase 1 and Phase 2.
2. Complete US1 backend (T019-T024), then US1 frontend (T025-T032).
3. Stop and validate the record-and-view journey.

### Incremental Delivery

1. Phase 1 + Phase 2 → foundation ready.
2. US1 backend → frontend → validate → demo.
3. US2 create/status balance work → frontend → validate its available paths.
4. US3, then US4; run T055 after US4 backend completes to validate every
   balance mutation path. Finish with US5 and validate each checkpoint.
5. Phase 8 verification before completion.

---

## Notes

- [P] tasks touch different files and have no incomplete dependency.
- Backend work stays in `../zunera-backend`; frontend work in
  `../zunera-frontend`; no new packages are added.
- Amounts are integer centavos only; never introduce floating point.
- Run focused tests after each task group and full suites before completion.
