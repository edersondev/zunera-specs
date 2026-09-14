# Tasks: Recurring Transactions

**Input**: Design documents from `specs/006-recurring-transactions/`  
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md),
[data-model.md](data-model.md), [contract](contracts/recurring-transactions-api.yaml),
[quickstart.md](quickstart.md)

**Tests**: Required. Every changed behavior gets backend feature/unit coverage;
contract changes get contract/feature coverage; frontend service/store/component
coverage and isolated Playwright critical journeys are required.

**Organization**: Tasks grouped by independently testable user story. For each
story, backend contract, authorization, validation, services, and tests complete
before frontend begins.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Parallelizable after prerequisites complete.
- **[Story]**: User story label. Setup, foundation, and polish omit it.
- Every task includes an exact target path.

## Phase 1: Setup

**Purpose**: Confirm coordinated worktree and contract baseline.

- [ ] T001 Verify `006-recurring-transactions` branch and clean intended scope in `/home/ederson/workspace/zunera/zunera-specs`, `../zunera-backend`, and `../zunera-frontend`
- [ ] T002 [P] Validate and preserve recurrence plus integrated transaction/history contracts with Redocly in `specs/006-recurring-transactions/contracts/recurring-transactions-api.yaml`, `specs/004-transaction-management/contracts/transactions-api.yaml`, and `specs/005-account-transfers/contracts/financial-history-api.yaml`
- [ ] T003 [P] Record planned recurrence translation keys and design-copy inventory in `specs/006-recurring-transactions/quickstart.md`

---

## Phase 2: Foundational

**Purpose**: Shared persistence and transaction-source prerequisites. Blocks all stories.

- [ ] T004 Create recurrence, recurrence-mutation-request, and transaction source-link migrations, including the recurrence `schedule_cursor` watermark column and its scheduling index, in `../zunera-backend/database/migrations/`
- [ ] T005 [P] Add recurrence enums, model, relationships, casts, scopes, and factory in `../zunera-backend/app/Enums/RecurringTransactions/`, `../zunera-backend/app/Models/RecurringTransaction.php`, and `../zunera-backend/database/factories/RecurringTransactionFactory.php`
- [ ] T006 [P] Extend transaction model, response data/resource, and financial-history resource with nullable `recurrence_source` in `../zunera-backend/app/Models/Transaction.php`, `../zunera-backend/app/Data/Transactions/TransactionResponseData.php`, `../zunera-backend/app/Http/Resources/Transactions/TransactionResource.php`, and `../zunera-backend/app/Http/Resources/FinancialHistory/FinancialHistoryResource.php`
- [ ] T007 [P] Add calendar, text-normalization, date-range, and recurrence idempotency services with unit tests in `../zunera-backend/app/Services/RecurringTransactions/` and `../zunera-backend/tests/Unit/RecurringTransactions/`
- [ ] T008 Add recurrence DTOs/filter data, Form Requests, API Resource, state exception, controller shell, and protected route registration in `../zunera-backend/app/Data/RecurringTransactions/`, `../zunera-backend/app/Exceptions/RecurringTransactions/`, `../zunera-backend/app/Http/{Requests,Resources}/RecurringTransactions/`, `../zunera-backend/app/Http/Controllers/Api/V1/RecurringTransactionController.php`, and `../zunera-backend/routes/api.php`
- [ ] T009 Add source-link regression tests for ordinary transaction detail/history compatibility in `../zunera-backend/tests/Feature/Transactions/RecurringTransactionSourceTest.php` and `../zunera-backend/tests/Feature/FinancialHistory/RecurringTransactionHistoryTest.php`

**Checkpoint**: Migration, source link, protected controller boundary, shared
calendar, and idempotency foundation exist. No user story starts before T004–T009.

---

## Phase 3: User Story 1 — Create a Recurring Income or Expense (Priority: P1) 🎯 MVP

**Goal**: Owner creates, lists, filters, and views valid weekly, monthly, or
yearly income/expense rule with next expected date.

**Independent Test**: User with active owned account and matching category saves
a rule and sees its persisted details/next date; invalid, foreign, archived, or
mismatched association cannot create it.

### Backend

- [ ] T010 [P] [US1] Write contract/feature tests for `GET` and `POST /recurring-transactions` in `../zunera-backend/tests/Feature/RecurringTransactions/CreateAndListRecurringTransactionsTest.php`
- [ ] T011 [P] [US1] Write validation/ownership/type-category/amount/date tests in `../zunera-backend/tests/Feature/RecurringTransactions/RecurringTransactionValidationTest.php`
- [ ] T012 [US1] Implement owned account/category resolution, create/list/filter, next-expected-date projection, and idempotent replay in `../zunera-backend/app/Services/RecurringTransactions/RecurringTransactionService.php`
- [ ] T013 [US1] Implement store/list request rules, controller responses, resource summaries, and create/list routes in `../zunera-backend/app/Http/{Requests,Controllers,Resources}/RecurringTransactions/` and `../zunera-backend/routes/api.php`
- [ ] T014 [US1] Run focused recurrence create/list/validation tests and contract lint from `../zunera-backend/tests/Feature/RecurringTransactions/` and `specs/006-recurring-transactions/contracts/recurring-transactions-api.yaml`

### Frontend — after T014

- [ ] T015 [P] [US1] Implement recurrence Axios list/create transport with idempotency handling in `../zunera-frontend/src/services/recurringTransactionService.js`
- [ ] T016 [P] [US1] Implement setup-style recurrence Pinia state, filters, error mapping, and retry keys in `../zunera-frontend/src/stores/recurring-transactions/recurringTransactionStore.js`
- [ ] T017 [P] [US1] Implement form dialog using active owned account/category choices and frequency/date validation in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionFormDialog.vue`
- [ ] T018 [P] [US1] Implement filter bar and scan-friendly list with state/type/next-date non-color labels in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionFilterBar.vue` and `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionList.vue`
- [ ] T019 [US1] Compose list/create route, authenticated route entry, navigation item, and localized labels in `../zunera-frontend/src/views/recurring-transactions/RecurringTransactionsListView.vue`, `../zunera-frontend/src/router/index.js`, `../zunera-frontend/src/layouts/AppShell.vue`, and `../zunera-frontend/src/i18n/messages.js`
- [ ] T020 [P] [US1] Add service/store/form/list unit tests in `../zunera-frontend/src/{services,stores,components}/**/__tests__/`
- [ ] T021 [US1] Add isolated create/list validation Playwright journey in `../zunera-frontend/e2e/recurring-transactions.spec.js`

**Checkpoint**: US1 works independently: valid rules create/list; invalid or
foreign data is safe; rule itself has no balance effect.

---

## Phase 4: User Story 2 — See Scheduled Occurrences in History (Priority: P1)

**Goal**: Due active rules produce exactly one pending existing transaction per
eligible date, visible with source origin in ordinary history.

**Independent Test**: Process same due rule repeatedly and concurrently; one
pending source-linked transaction appears, account balance stays unchanged, and
owner can identify source from rule/history/detail.

### Backend

- [ ] T022 [P] [US2] Write unit tests for weekly/monthly/yearly dates, short months, leap years, creation anchor, paused skip, catch-up, and end-date behavior in `../zunera-backend/tests/Unit/RecurringTransactions/RecurringScheduleCalculatorTest.php`
- [ ] T023 [P] [US2] Write feature/concurrency tests for pending occurrence creation, duplicate rule/date prevention, retries, balance-zero effect, and owner-scoped paginated occurrence listing in `../zunera-backend/tests/Feature/RecurringTransactions/ProcessRecurringOccurrencesTest.php`
- [ ] T024 [US2] Implement schedule calculator and locked occurrence processor that creates pending source-linked transactions, advances the schedule cursor, never evaluates paused or pre-eligibility dates, and auto-ends expired rules in `../zunera-backend/app/Services/RecurringTransactions/{RecurringScheduleCalculator,RecurringOccurrenceService}.php`
- [ ] T025 [US2] Register due-processing command and application scheduler entry in `../zunera-backend/app/Console/Commands/ProcessRecurringTransactions.php` and `../zunera-backend/routes/console.php`
- [ ] T026 [US2] Implement owner-scoped paginated generated-occurrence listing and confirm T006 source serialization still returns `recurrence_source` for generated occurrences in `../zunera-backend/app/Services/RecurringTransactions/RecurringTransactionService.php`, `../zunera-backend/app/Http/Controllers/Api/V1/RecurringTransactionController.php`, `../zunera-backend/app/Http/Resources/RecurringTransactions/GeneratedOccurrenceResource.php`, and `../zunera-backend/routes/api.php`
- [ ] T027 [US2] Run due-processing, transaction-history, and financial-history regression suites in `../zunera-backend/tests/{Unit,Feature}/`

### Frontend — after T027

- [ ] T028 [P] [US2] Add recurrence-source formatter and source label/link UI to transaction detail/history rows in `../zunera-frontend/src/utils/recurring-transactions/recurringTransactionFormatters.js`, `../zunera-frontend/src/components/transactions/TransactionDetailDrawer.vue`, and `../zunera-frontend/src/views/transactions/TransactionsListView.vue`
- [ ] T029 [P] [US2] Implement paginated occurrence transport/state in `../zunera-frontend/src/services/recurringTransactionService.js` and `../zunera-frontend/src/stores/recurring-transactions/recurringTransactionStore.js`; implement recurrence detail drawer that consumes this state and navigates to ordinary transaction details in `../zunera-frontend/src/components/recurring-transactions/RecurringTransactionDetailDrawer.vue`
- [ ] T030 [P] [US2] Add source-label/detail and pending/no-balance frontend unit tests in `../zunera-frontend/src/{components,utils}/**/__tests__/`
- [ ] T031 [US2] Extend `../zunera-frontend/e2e/recurring-transactions.spec.js` with due, catch-up, source-origin, and pending-confirmation journey

**Checkpoint**: US2 works independently: recurrence and ordinary transaction
history remain one explainable system with no duplicate or automatic balance effect.

---

## Phase 5: User Story 3 — Manage Future Recurrence Rules (Priority: P2)

**Goal**: Owner updates future rule information, pauses/resumes, and ends rules
without rewriting generated history.

**Independent Test**: Edit affects only ungenerated dates; pause skips dates;
resume does not backfill; archive pauses; end is terminal and preserved history.

### Backend

- [ ] T032 [P] [US3] Write update/pause/resume/end contract, state-conflict, start/end date edit re-anchor, no next-expected-date while paused, cursor-on-resume no-backfill, and terminal-state tests in `../zunera-backend/tests/Feature/RecurringTransactions/ManageRecurringTransactionsTest.php`
- [ ] T033 [P] [US3] Write archive-trigger and historical-snapshot tests in `../zunera-backend/tests/Feature/RecurringTransactions/ArchivedAssociationRecurringTransactionsTest.php`
- [ ] T034 [US3] Implement update scope, start/end date re-anchor rules, lifecycle state machine, association-repair restriction, idempotent actions, terminal-state rejection, and schedule-cursor advance on resume in `../zunera-backend/app/Services/RecurringTransactions/RecurringTransactionService.php`
- [ ] T035 [US3] Invoke automatic recurrence pause from account/category archive services in `../zunera-backend/app/Services/{FinancialAccounts,Categories}/` and retain archived summaries in recurrence resources
- [ ] T036 [US3] Implement patch/pause/resume/end requests, controller methods, and routes in `../zunera-backend/app/Http/{Requests,Controllers}/RecurringTransactions/` and `../zunera-backend/routes/api.php`
- [ ] T037 [US3] Run lifecycle/archive/history regression suites and recurrence contract lint in `../zunera-backend/tests/Feature/RecurringTransactions/` and `specs/006-recurring-transactions/contracts/recurring-transactions-api.yaml`

### Frontend — after T037

- [ ] T038 [P] [US3] Add update/pause/resume/end service and Pinia mutation actions with durable refresh/error states in `../zunera-frontend/src/{services,stores}/recurring-transactions/`
- [ ] T039 [P] [US3] Implement lifecycle confirmation dialog and row actions with archived-association repair explanation in `../zunera-frontend/src/components/recurring-transactions/{RecurringTransactionLifecycleDialog,RecurringTransactionRowActions}.vue`
- [ ] T040 [US3] Wire edit/detail/lifecycle actions and active/paused/ended feedback into `../zunera-frontend/src/views/recurring-transactions/RecurringTransactionsListView.vue`
- [ ] T041 [P] [US3] Add lifecycle store/component unit tests in `../zunera-frontend/src/{stores,components}/recurring-transactions/__tests__/`
- [ ] T042 [US3] Extend `../zunera-frontend/e2e/recurring-transactions.spec.js` with edit, pause, resume, archive repair, and end journeys

**Checkpoint**: US3 works independently: future rule changes and lifecycle
actions preserve historical occurrence snapshots and never backfill paused dates.

---

## Phase 6: User Story 4 — Correct One Occurrence (Priority: P2)

**Goal**: Owner corrects one generated transaction while source rule and other
occurrences retain their scheduled values.

**Independent Test**: Change April generated transaction, verify rule/May
unchanged, and clearly see difference between one occurrence versus rule edit.

### Backend

- [ ] T043 [P] [US4] Add generated-occurrence update/remove/restore isolation tests in `../zunera-backend/tests/Feature/RecurringTransactions/IndividualOccurrenceCorrectionTest.php`
- [ ] T044 [US4] Enforce source-link immutability during ordinary transaction update/remove/restore and preserve recurrence rule/sibling snapshots in `../zunera-backend/app/Services/Transactions/TransactionService.php` and `../zunera-backend/app/Models/Transaction.php`
- [ ] T045 [US4] Run transaction and individual-occurrence regression tests in `../zunera-backend/tests/{Feature/Transactions,Feature/RecurringTransactions}/`

### Frontend — after T045

- [ ] T046 [P] [US4] Add explicit occurrence-versus-rule editing guidance and source-aware detail actions in `../zunera-frontend/src/components/{transactions,recurring-transactions}/`
- [ ] T047 [P] [US4] Add source-aware transaction detail/action unit tests in `../zunera-frontend/src/components/transactions/__tests__/TransactionDetailDrawer.spec.js`
- [ ] T048 [US4] Extend `../zunera-frontend/e2e/recurring-transactions.spec.js` with one-occurrence correction regression journey

**Checkpoint**: US4 works independently: exceptional occurrence correction does
not mutate recurrence rule or later transactions.

---

## Phase 7: User Story 5 — Find and Understand Recurrences (Priority: P3)

**Goal**: Owner finds rules by type, account, category, frequency, and state and
understands amount, schedule, association, status, and next date.

**Independent Test**: Combined filters return only matching owned rules; no-match
state and clear criteria work across active, paused, and ended records.

### Backend

- [ ] T049 [P] [US5] Add combined filter, owner-scope, pagination, ordering, and seeded 5,000-rule scale tests that assert the first 50 matching rules return in under 2 seconds in `../zunera-backend/tests/Feature/RecurringTransactions/FilterRecurringTransactionsTest.php` and `../zunera-backend/tests/Feature/RecurringTransactions/RecurringTransactionScaleTest.php`
- [ ] T050 [US5] Complete filter validation, AND query behavior, next-date ordering with null next-expected-date semantics for paused and ended rules, count metadata, and archived association filter resolution in `../zunera-backend/app/Services/RecurringTransactions/RecurringTransactionService.php` and `../zunera-backend/app/Http/Requests/RecurringTransactions/ListRecurringTransactionsRequest.php`
- [ ] T051 [US5] Run filter/scale tests and confirmed list-contract validation in `../zunera-backend/tests/Feature/RecurringTransactions/` and `specs/006-recurring-transactions/contracts/recurring-transactions-api.yaml`

### Frontend — after T051

- [ ] T052 [US5] Add route-query persistence, combined criteria labels, pagination/load-more, loading/error/no-match states, and clear action in `../zunera-frontend/src/views/recurring-transactions/RecurringTransactionsListView.vue` and `../zunera-frontend/src/stores/recurring-transactions/recurringTransactionStore.js`
- [ ] T053 [P] [US5] Add filter/list/store empty-state and responsive-column unit tests in `../zunera-frontend/src/{components,stores,views}/recurring-transactions/__tests__/`
- [ ] T054 [US5] Extend `../zunera-frontend/e2e/recurring-transactions.spec.js` with combined filters, no-match, active criteria, and next-date discovery journey

**Checkpoint**: US5 works independently: user rapidly locates and interprets
matching recurrence rules without cross-user disclosure.

---

## Phase 8: Polish and Cross-Cutting Verification

**Purpose**: Complete accessibility, privacy, contract, performance, and full
feature verification after all stories.

- [ ] T055 [P] Add cross-user, idempotency-key reuse, invalid-state, and privacy-safe response regression coverage in `../zunera-backend/tests/Feature/RecurringTransactions/`
- [ ] T056 [P] Add keyboard, focus return, 320px, 200% zoom, light/dark/system, and non-color source/state coverage in `../zunera-frontend/e2e/recurring-transactions.spec.js`
- [ ] T057 Run backend migrations, full tests, route check, and Pint in `../zunera-backend/`
- [ ] T058 Run Redocly lint for `specs/006-recurring-transactions/contracts/recurring-transactions-api.yaml`, `specs/004-transaction-management/contracts/transactions-api.yaml`, and `specs/005-account-transfers/contracts/financial-history-api.yaml`
- [ ] T059 Run frontend unit suite, production build, and isolated Playwright feature suite in `../zunera-frontend/`
- [ ] T060 Update implementation evidence in `specs/006-recurring-transactions/quickstart.md`

## Dependencies and Execution Order

```text
Setup → Foundational → US1 → US2 → US3 → US4 → US5 → Polish
                         └───────────────────────────────┘
```

- US1 needs foundational persistence/contract and is MVP.
- US2 needs US1 rule creation plus source-link foundation.
- US3 needs US1 rule persistence; it may follow US2, preserving generated data.
- US4 needs US2 generated occurrences and existing transaction management.
- US5 needs US1 list/store and can complete after US3 state behavior exists.
- Every frontend block depends on its preceding backend checkpoint.

## Parallel Opportunities

- T003 and T005–T007/T009 use independent files after migration design.
- Within US1: T010/T011 and T015–T018/T020 can run in parallel at their stated gates.
- Within US2: T022/T023 and T028–T030 can run in parallel.
- Within US3: T032/T033 and T038/T039/T041 can run in parallel.
- Within US4: T046/T047 can run in parallel after T045.
- Within US5: T053 can run alongside backend completion once UI contract is stable.
- T055/T056 can run in parallel after all story checkpoints.

## Implementation Strategy

### MVP First

1. Complete setup and foundation.
2. Complete US1 backend through T014, then US1 frontend through T021.
3. Validate creation/listing, ownership, associations, and zero direct balance
   effect before continuing.

### Incremental Delivery

1. Add US2 to make rules generate explainable pending history.
2. Add US3 lifecycle and association archive protection.
3. Add US4 exceptional occurrence correction boundary.
4. Add US5 scalable organization/discovery.
5. Finish cross-cutting verification and quickstart evidence.

### Format Validation

All 60 tasks use checkbox, sequential task ID, optional `[P]`, required story
label only in story phases, and an exact target path — or the repository root
for command-only verification tasks such as T057 and T059.
