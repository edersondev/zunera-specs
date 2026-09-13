# Tasks: Account Transfers

**Input**: Design documents from specs/005-account-transfers  
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/,
quickstart.md  
**Tests**: Required by SQR-002 and constitution: Laravel unit/feature, OpenAPI
contract lint, frontend unit, and isolated Playwright coverage.  
**Organization**: Tasks grouped by user story. Backend work completes before its
frontend work.

## Format: [ID] [P?] [Story] Description

- [P] means different file with no incomplete dependency.
- [USn] maps task to a spec user story.
- No new packages. Use integer centavos only.

## Phase 1: Setup

**Purpose**: Confirm coordinated workspaces and create domain locations.

- [ ] T001 Verify branch 005-account-transfers in ., ../zunera-backend, and ../zunera-frontend before application edits.
- [ ] T002 [P] Create transfer backend folders in ../zunera-backend/app/{Data,Enums,Exceptions,Http/Requests,Http/Resources,Services}/Transfers and ../zunera-backend/tests/{Feature,Unit}/Transfers.
- [ ] T003 [P] Create transfer frontend folders in ../zunera-frontend/src/{components,stores,utils,views}/transfers and ../zunera-frontend/e2e without adding dependencies.

---

## Phase 2: Foundational

**Purpose**: Shared transfer primitives. Complete before every story.

- [ ] T004 [P] Create TransferStatus enum with pending and effective in ../zunera-backend/app/Enums/Transfers/TransferStatus.php.
- [ ] T005 [P] Create positive centavo bounds and BRL parsing helper in ../zunera-backend/app/Services/Transfers/TransferMoney.php.
- [ ] T006 [P] Create ISO/Brazilian date parser, 1900-01-01..2100-12-31 bounds, and future-date helper in ../zunera-backend/app/Services/Transfers/TransferDateRange.php.
- [ ] T007 [P] Create case/accent-insensitive description/notes normalizer in ../zunera-backend/app/Services/Transfers/TransferTextNormalizer.php.
- [ ] T008 [P] Create typed transfer state exception codes in ../zunera-backend/app/Exceptions/Transfers/TransferStateException.php.
- [ ] T009 Create transfers table with owner, distinct source/destination associations, status, exact money, optional text, search text, removal state, timestamps, and history indexes in ../zunera-backend/database/migrations/*_create_transfers_table.php.
- [ ] T010 Create owner-scoped transfer mutation request table with key, operation, fingerprint, target, replay status/body, and unique owner/key constraint in ../zunera-backend/database/migrations/*_create_transfer_mutation_requests_table.php.
- [ ] T011 Create guarded Transfer model, casts, source/destination/user relations, search-text update hook, and active/removed/counting scopes in ../zunera-backend/app/Models/Transfer.php.
- [ ] T012 [P] Create TransferMutationRequest model with owner/key replay relation in ../zunera-backend/app/Models/TransferMutationRequest.php.
- [ ] T013 [P] Create TransferFactory states for pending, effective, removed, future, and archived-side history in ../zunera-backend/database/factories/TransferFactory.php.
- [ ] T014 [P] Create create, update, and filter DTOs in ../zunera-backend/app/Data/Transfers/CreateTransferData.php, UpdateTransferData.php, and TransferFilterData.php.
- [ ] T015 [P] Create list, store, update, and lifecycle request skeletons with mutation idempotency-header validation in ../zunera-backend/app/Http/Requests/Transfers/ListTransfersRequest.php, StoreTransferRequest.php, UpdateTransferRequest.php, and LifecycleTransferRequest.php.
- [ ] T016 Implement deterministic two-account delta reconciliation with proposed source/destination balance validation and ascending account locks in ../zunera-backend/app/Services/Transfers/TransferBalanceReconciler.php.
- [ ] T017 Implement owner-scoped mutation key acquisition, exact replay, and changed-payload conflict behavior in ../zunera-backend/app/Services/Transfers/TransferIdempotencyService.php.
- [ ] T018 Create TransferResource with source/destination embedded account summaries, lifecycle status, and future-date notice metadata in ../zunera-backend/app/Http/Resources/Transfers/TransferResource.php.
- [ ] T019 Register transfer CRUD/lifecycle routes inside existing authentication and session middleware in ../zunera-backend/routes/api.php.
- [ ] T020 Create thin TransferController index, store, show, update, remove, and restore actions in ../zunera-backend/app/Http/Controllers/Api/V1/TransferController.php.
- [ ] T021 [P] Lint transfer contract and record result in specs/005-account-transfers/contracts/transfers-api.yaml and specs/005-account-transfers/quickstart.md.

**Checkpoint**: Persistence, service primitives, resource, routes, and contract
exist. Story work may begin.

---

## Phase 3: User Story 1 - Record an Account Transfer (Priority: P1) MVP

**Goal**: Record pending/effective transfer between two active owned accounts
without creating, destroying, or overdrawing money.

**Independent Test**: Create R$ 1.000,00 effective transfer from R$ 5.000,00
source to R$ 2.000,00 destination; verify R$ 4.000,00/R$ 3.000,00, unchanged
combined balance, ownership and validation denials, and future pending behavior.

### Tests for User Story 1

- [ ] T022 [P] [US1] Add creation feature tests for owned active sides, same-side rejection, amount/date/text validation, and privacy-safe foreign account denial in ../zunera-backend/tests/Feature/Transfers/RecordTransfersTest.php.
- [ ] T023 [P] [US1] Add unit tests for amount/date rules, pending future default, explicit future-effective rejection, and no-reservation pending behavior in ../zunera-backend/tests/Unit/Transfers/TransferRulesTest.php.
- [ ] T024 [P] [US1] Add reconciler unit matrix for source debit, destination credit, exact combined total, source overdraft rejection, destination bounds, and deterministic locks in ../zunera-backend/tests/Unit/Transfers/TransferBalanceReconcilerTest.php.

### Implementation for User Story 1

#### Backend

- [ ] T025 [US1] Implement TransferService create flow: owner/active/distinct-side validation, future-status rule, no-reservation pending rule, history flag, idempotency, and atomic reconciliation in ../zunera-backend/app/Services/Transfers/TransferService.php.
- [ ] T026 [US1] Implement store request field rules, DTO conversion, active owned side resolution, and privacy-safe foreign-side mapping in ../zunera-backend/app/Http/Requests/Transfers/StoreTransferRequest.php.
- [ ] T027 [US1] Wire created-resource and typed validation/conflict responses in ../zunera-backend/app/Http/Controllers/Api/V1/TransferController.php.
- [ ] T028 [US1] Run focused creation/reconciler tests and Pint in ../zunera-backend/tests/{Feature,Unit}/Transfers and ../zunera-backend.

#### Frontend (after T028)

- [ ] T029 [P] [US1] Implement create, list, and get requests with one idempotency key per logical create in ../zunera-frontend/src/services/transferService.js.
- [ ] T030 [US1] Implement setup-style transfer store with active list, active transfer, create action, in-flight guard, and account-refresh hook in ../zunera-frontend/src/stores/transfers/transferStore.js.
- [ ] T031 [US1] Implement BRL, date, status, and Source → Destination formatters in ../zunera-frontend/src/utils/transfers/transferFormatters.js and ../zunera-frontend/src/utils/transfers/transferOptions.js.
- [ ] T032 [US1] Build create form with active owned accounts, source exclusion from destination, BRL amount, date, status, optional text, server errors, and duplicate-submit guard in ../zunera-frontend/src/components/transfers/TransferFormDialog.vue.
- [ ] T033 [US1] Build basic newest-first transfer list with source, destination, amount, date, status, and non-color transfer label in ../zunera-frontend/src/views/transfers/TransfersListView.vue.
- [ ] T034 [US1] Add authenticated Transfers route, navigation item, and locale keys in ../zunera-frontend/src/router/index.js, ../zunera-frontend/src/composables/, and ../zunera-frontend/src/i18n/.
- [ ] T035 [P] [US1] Add service, store, formatter, and form unit tests for payloads, pending no-reservation messaging, source/destination exclusion, and submit guard in ../zunera-frontend/src/services/__tests__/transferService.spec.js, ../zunera-frontend/src/stores/transfers/__tests__/transferStore.spec.js, ../zunera-frontend/src/utils/transfers/__tests__/transferFormatters.spec.js, and ../zunera-frontend/src/components/transfers/__tests__/TransferFormDialog.spec.js.
- [ ] T036 [US1] Add isolated create effective/pending, validation, and both-balance Playwright journey in ../zunera-frontend/e2e/transfers.spec.js.
- [ ] T037 [US1] Run frontend unit, build, and focused transfer journey commands in ../zunera-frontend/package.json and ../zunera-frontend/e2e/transfers.spec.js.

**Checkpoint**: User can create a valid transfer and observe complete two-account
effect independently.

---

## Phase 4: User Story 2 - Review and Find Transfers (Priority: P2)

**Goal**: Users list, inspect, filter, and search owned transfers.

**Independent Test**: With multiple transfers, filter by date/source/destination/
status, search accent-varied text, open detail, load more results, and confirm
foreign data never appears.

### Tests for User Story 2

- [ ] T038 [P] [US2] Add transfer history/list tests for ordering, pagination cap, total count, active/removed views, detail, owner isolation, and unauthenticated denial in ../zunera-backend/tests/Feature/Transfers/ListTransferHistoryTest.php and ../zunera-backend/tests/Feature/Transfers/ViewTransferDetailsTest.php.
- [ ] T039 [P] [US2] Add filter/search tests for ISO/Brazilian dates, source/destination/status filters, archived filter sides, foreign filter 404, AND semantics, and accent-insensitive text in ../zunera-backend/tests/Feature/Transfers/FilterAndSearchTransfersTest.php.
- [ ] T040 [P] [US2] Add a 5,000-transfer scale test using documented seeded test data that verifies newest 50, exact total, progressive reachability, stable same-date ordering, and elapsed first-batch response time of no more than two seconds in ../zunera-backend/tests/Feature/Transfers/TransferHistoryScaleTest.php and specs/005-account-transfers/quickstart.md.

### Implementation for User Story 2

#### Backend

- [ ] T041 [US2] Implement list/filter/search DTO parsing, date range validation, active/removed view, and page size cap in ../zunera-backend/app/Http/Requests/Transfers/ListTransfersRequest.php and ../zunera-backend/app/Data/Transfers/TransferFilterData.php.
- [ ] T042 [US2] Implement owner-scoped TransferService list and findOwned queries with embedded archived summaries, normalized search, filters, pagination meta, and stable newest-first ordering in ../zunera-backend/app/Services/Transfers/TransferService.php.
- [ ] T043 [US2] Wire index/show resources and privacy-safe account filter behavior in ../zunera-backend/app/Http/Controllers/Api/V1/TransferController.php.
- [ ] T044 [US2] Run focused history, filter/search, scale, and contract tests in ../zunera-backend/tests/Feature/Transfers and specs/005-account-transfers/contracts/transfers-api.yaml.

#### Frontend (after T044)

- [ ] T045 [US2] Add list/detail/filter request mapping, pagination links, and query serialization in ../zunera-frontend/src/services/transferService.js.
- [ ] T046 [US2] Extend shared transfer list/filter/page/meta state and route-query synchronization in ../zunera-frontend/src/stores/transfers/transferStore.js.
- [ ] T047 [US2] Build date, source, destination, status, and text filter controls with visible active criteria and reset in ../zunera-frontend/src/components/transfers/TransferFilterBar.vue.
- [ ] T048 [US2] Add progressive loading, matching count, loading/error/empty/no-match/retry states in ../zunera-frontend/src/views/transfers/TransfersListView.vue.
- [ ] T049 [US2] Build transfer detail drawer with both account summaries, archived labels, status, optional text, and focus restoration in ../zunera-frontend/src/components/transfers/TransferDetailDrawer.vue.
- [ ] T050 [P] [US2] Add list/filter/detail/store unit tests for route query, pagination, state displays, archived labels, and normalized search in ../zunera-frontend/src/components/transfers/__tests__/TransferFilterBar.spec.js, ../zunera-frontend/src/components/transfers/__tests__/TransferDetailDrawer.spec.js, ../zunera-frontend/src/views/transfers/__tests__/TransfersListView.spec.js, and ../zunera-frontend/src/stores/transfers/__tests__/transferStore.spec.js.
- [ ] T051 [US2] Extend transfer Playwright coverage for history/detail/filter/search/clear/no-match/mobile keyboard flow in ../zunera-frontend/e2e/transfers.spec.js.

**Checkpoint**: User can find and inspect any owned transfer independently.

---

## Phase 5: User Story 3 - Correct, Remove, and Restore a Transfer (Priority: P2)

**Goal**: User corrects both-sided movement safely, removes it recoverably, and
restores it without stale or duplicate balance effects.

**Independent Test**: Edit sides, amount, date, status, text; retain archived
side; remove and restore; verify source/destination balances every time.

### Tests for User Story 3

- [ ] T052 [P] [US3] Add update tests for side/amount/date/status/text edits, archived-side retain-versus-replace rules, foreign 404, removed-edit conflict, effective-transfer retimed-future notice with retained effect, pending-future explicit-effective 422 effective_future_date rejection, and proposed-balance overdraft validation in ../zunera-backend/tests/Feature/Transfers/UpdateTransfersTest.php.
- [ ] T053 [P] [US3] Add remove/restore tests for both-side reversal, removed view, repeated lifecycle conflicts, future restore default, explicit future-effective restore 422 effective_future_date rejection, funds recheck, idempotent replay, and changed-key conflict in ../zunera-backend/tests/Feature/Transfers/RemoveAndRestoreTransfersTest.php.
- [ ] T054 [P] [US3] Add full mutation balance consistency tests for multi-account moves, pending/effective changes, removal/restoration, and account summary refresh in ../zunera-backend/tests/Feature/Transfers/TransferBalanceConsistencyTest.php.

### Implementation for User Story 3

#### Backend

- [ ] T055 [US3] Implement update and lifecycle validation, archived current-side retention, replacement-side rejection, pending-future explicit-effective 422 mapping, idempotency header, and DTO conversion in ../zunera-backend/app/Http/Requests/Transfers/UpdateTransferRequest.php and ../zunera-backend/app/Http/Requests/Transfers/LifecycleTransferRequest.php.
- [ ] T056 [US3] Implement TransferService update with old/new effects, all affected account locks, change detection, proposed balances, status/date notice, and idempotency replay in ../zunera-backend/app/Services/Transfers/TransferService.php.
- [ ] T057 [US3] Implement TransferService remove/restore with immutable removed state, future-effective restore rejection, funds recheck, state conflicts, and exactly-once two-side reconciliation in ../zunera-backend/app/Services/Transfers/TransferService.php.
- [ ] T058 [US3] Wire update/remove/restore typed responses and resource metadata in ../zunera-backend/app/Http/Controllers/Api/V1/TransferController.php.
- [ ] T059 [US3] Run focused update/lifecycle/balance tests and Pint in ../zunera-backend/tests/{Feature,Unit}/Transfers and ../zunera-backend.

#### Frontend (after T059)

- [ ] T060 [US3] Implement update/remove/restore requests with per-action idempotency retry in ../zunera-frontend/src/services/transferService.js.
- [ ] T061 [US3] Implement update/remove/restore actions, account/history refresh, typed conflict feedback, and mutation in-flight states in ../zunera-frontend/src/stores/transfers/transferStore.js.
- [ ] T062 [US3] Extend form for editing, archived retained sides, future-effective notice, and correct action feedback in ../zunera-frontend/src/components/transfers/TransferFormDialog.vue.
- [ ] T063 [US3] Build Removed transfer view and Restore confirmation flow in ../zunera-frontend/src/views/transfers/RemovedTransfersView.vue and ../zunera-frontend/src/components/transfers/TransferLifecycleConfirmDialog.vue.
- [ ] T064 [US3] Add Removed route, navigation/context links, and locale keys in ../zunera-frontend/src/router/index.js, ../zunera-frontend/src/composables/, and ../zunera-frontend/src/i18n/.
- [ ] T065 [P] [US3] Add update/lifecycle/store component tests for idempotency reuse, archived sides, date notice, removed state, and account refresh in ../zunera-frontend/src/services/__tests__/transferService.spec.js, ../zunera-frontend/src/stores/transfers/__tests__/transferStore.spec.js, and ../zunera-frontend/src/components/transfers/__tests__/TransferFormDialog.spec.js.
- [ ] T066 [US3] Extend transfer Playwright coverage for edit, side move, status, remove, restore, archived association, and balance feedback in ../zunera-frontend/e2e/transfers.spec.js.

**Checkpoint**: All correction and lifecycle actions preserve two-account
integrity independently.

---

## Phase 6: User Story 4 - Distinguish Transfers from Income and Expenses (Priority: P3)

**Goal**: Existing financial/transaction history displays labelled transfers
while reports and income/expense totals exclude them.

**Independent Test**: Mixed history shows a Source → Destination transfer beside
income/expense; before/after totals and reports stay unchanged by transfer.

### Tests for User Story 4

- [ ] T067 [P] [US4] Add mixed-history feature/contract tests for the canonical GET /financial-history route: movement discriminator, transfer account sides, owner isolation, account filter matching either side, pagination, transfer-free income/expense totals, and no duplicate GET /transactions registration in ../zunera-backend/tests/Feature/FinancialHistory/ListFinancialHistoryTest.php and specs/005-account-transfers/contracts/financial-history-api.yaml.
- [ ] T068 [P] [US4] Add named aggregate regression tests proving FinancialHistoryService excludes effective transfers from income, expense, and financial-result totals, while FinancialAccountService applies both side effects and preserves combined owned-account net worth, in ../zunera-backend/tests/Feature/FinancialHistory/TransferReportingExclusionTest.php.

### Implementation for User Story 4

#### Backend

- [ ] T069 [US4] Implement discriminated income, expense, and transfer history projection with transfer account summaries and no category in ../zunera-backend/app/Services/FinancialHistory/FinancialHistoryService.php and ../zunera-backend/app/Http/Resources/FinancialHistory/FinancialHistoryResource.php.
- [ ] T070 [US4] Implement the sole authenticated GET /financial-history mixed-history endpoint with movement-kind/account/status/date/search filters and privacy-safe errors in ../zunera-backend/app/Http/Controllers/Api/V1/FinancialHistoryController.php and ../zunera-backend/routes/api.php; retain the feature 004 transaction-only GET /transactions route without a duplicate registration.
- [ ] T071 [US4] Restrict FinancialHistoryService income, expense, and financial-result aggregates to income/expense movements; make FinancialAccountService apply both account effects and derive combined owned-account net worth from both sides in ../zunera-backend/app/Services/FinancialHistory/FinancialHistoryService.php and ../zunera-backend/app/Services/FinancialAccounts/FinancialAccountService.php.
- [ ] T072 [US4] Run mixed-history/report tests and lint financial-history contract in ../zunera-backend/tests/Feature/FinancialHistory and specs/005-account-transfers/contracts/financial-history-api.yaml.

#### Frontend (after T072)

- [ ] T073 [US4] Extend financial/transaction history service and state to consume discriminated movement entries from canonical GET /financial-history in ../zunera-frontend/src/services/transactionService.js and ../zunera-frontend/src/stores/transactions/transactionStore.js.
- [ ] T074 [US4] Render transfer history rows as explicitly labelled Source → Destination movements, with neutral icon/text and no income/expense sign or total impact, in ../zunera-frontend/src/views/transactions/TransactionsListView.vue and ../zunera-frontend/src/components/transactions/TransactionDetailDrawer.vue.
- [ ] T075 [P] [US4] Add mixed-history renderer/store tests for movement discrimination, account-side text, non-color distinction, and unchanged income/expense totals in ../zunera-frontend/src/views/transactions/__tests__/TransactionsListView.spec.js and ../zunera-frontend/src/stores/transactions/__tests__/transactionStore.spec.js.
- [ ] T076 [US4] Extend transfer Playwright journey for mixed history, report exclusion, theme, zoom, and keyboard discrimination checks in ../zunera-frontend/e2e/transfers.spec.js.

**Checkpoint**: Transfers appear in existing history but remain financially and
visually distinct from income/expense.

---

## Phase 7: Polish and Cross-Cutting Validation

**Purpose**: Complete feature-wide checks and evidence.

- [ ] T077 [P] Re-lint both OpenAPI contracts and resolve response-shape drift in specs/005-account-transfers/contracts/transfers-api.yaml and specs/005-account-transfers/contracts/financial-history-api.yaml.
- [ ] T078 [P] Reconcile final implementation decisions and verification evidence in specs/005-account-transfers/{research.md,data-model.md,quickstart.md}.
- [ ] T079 [P] Run full backend tests and style validation in ../zunera-backend using php artisan test --compact and vendor/bin/pint --dirty --format agent.
- [ ] T080 [P] Run frontend unit suite and production build in ../zunera-frontend using npm run test:unit -- --run and npm run build.
- [ ] T081 Run isolated transfer Playwright suite across configured browsers in ../zunera-frontend/e2e/transfers.spec.js.
- [ ] T082 Verify light/dark/system, 320px, 200% zoom, keyboard-only, dialog/drawer focus restoration, and non-color transfer distinction in ../zunera-frontend/src/{views,components}/transfers.
- [ ] T083 Run manual smoke checklist, record exact command/test evidence, and record the seeded 5,000-transfer first-batch elapsed time against the two-second budget in specs/005-account-transfers/quickstart.md.
- [ ] T084 Run cross-artifact consistency analysis after task completion using specs/005-account-transfers/{spec.md,plan.md,tasks.md}.
- [ ] T085 Commit reviewed feature artifacts from specs/005-account-transfers and AGENTS.md using project git workflow.

---

## Dependencies and Execution Order

### Phase Dependencies

- Setup: no dependencies.
- Foundational: after Setup; blocks all stories.
- US1: after Foundational; MVP.
- US2: after US1 create contract exists.
- US3: after US1 create path; works with US2 history.
- US4: after US1 transfer model and US2 history work.
- Polish: after desired stories complete.

### User Story Dependencies

- **US1**: Independent MVP after foundation.
- **US2**: Uses transfer list/detail API from foundation and transfer records from
  US1.
- **US3**: Uses US1 transfer creation; does not require mixed history.
- **US4**: Uses transfers from US1 and existing transaction-history UI.

### Within Each Story

- Backend tests and contract first, then backend implementation and focused
  verification.
- Frontend starts only after backend API, authorization, validation, and tests.
- Service before store; store before component/view; component tests before
  story checkpoint.

## Parallel Opportunities

- Setup T002-T003; foundation T004-T008, T012-T015, and T021.
- Tests T022-T024, T038-T040, T052-T054, and T067-T068 within each story.
- Frontend tasks marked [P] do not overlap their story implementation files.
- US2 and US3 backend work may proceed concurrently after US1 backend checkpoint
  when separate developers avoid shared TransferService merge conflicts.

## Parallel Example: User Story 1

    Task: T022 RecordTransfersTest.php
    Task: T023 TransferRulesTest.php
    Task: T024 TransferBalanceReconcilerTest.php

    Task: T029 transferService.js
    Task: T031 transferFormatters.js and transferOptions.js

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete US1 backend T022-T028, then frontend T029-T037.
3. Stop and validate the create/pending/overdraft/balance journey.

### Incremental Delivery

1. Add US2 transfer history and discovery; verify independently.
2. Add US3 corrections and removal; verify every balance mutation.
3. Add US4 mixed financial history and reporting exclusion.
4. Complete Phase 7 before feature handoff.

## Notes

- All 85 tasks follow checkbox, ID, optional parallel marker, story label where
  applicable, and exact path format.
- Backend and frontend branches must remain 005-account-transfers.
- Never introduce floating-point money or permanent transfer deletion.
