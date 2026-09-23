# Tasks: Transactions Activity Redesign

**Input**: [spec.md](spec.md), [plan.md](plan.md), [research.md](research.md), [history UI contract](contracts/history-ui.md)
**Delivery order**: Existing backend contracts already verified; frontend only. Tests precede corresponding behavior.

## Phase 1: Setup

- [x] T001 Verify matching specs/frontend branches and existing backend contracts in specs/013-transactions-activity-redesign/plan.md
- [x] T002 Review Design Foundation and existing Transactions, Budgets, and Statement Details components in specs/013-transactions-activity-redesign/research.md

## Phase 2: Foundational

- [x] T003 Add shared month-navigation tests in ../zunera-frontend/src/components/budgets/__tests__/BudgetMonthNavigator.spec.js and transaction-period tests in ../zunera-frontend/src/utils/transactions/__tests__/transactionPeriod.spec.js
- [x] T004 Extract shared month navigator into ../zunera-frontend/src/components/common/MonthNavigator.vue and preserve Budgets adapter in ../zunera-frontend/src/components/budgets/BudgetMonthNavigator.vue
- [x] T005 Implement full-month/custom period helpers in ../zunera-frontend/src/utils/transactions/transactionPeriod.js

## Phase 3: User Story 1 - Browse monthly activity (P1)

**Independent Test**: Month/year changes update list, URL, grouping, and page one.

- [x] T006 [US1] Add route, month, and grouping tests in ../zunera-frontend/src/views/transactions/__tests__/TransactionsListView.spec.js and ../zunera-frontend/src/components/transactions/__tests__/TransactionHistoryList.spec.js
- [x] T007 [US1] Coordinate route-backed period and store refresh in ../zunera-frontend/src/views/transactions/TransactionsListView.vue and ../zunera-frontend/src/stores/transactions/transactionStore.js
- [x] T008 [US1] Replace table/mobile split with date groups in ../zunera-frontend/src/components/transactions/TransactionHistoryList.vue

## Phase 4: User Story 2 - Inspect and act on a movement (P2)

**Independent Test**: Every movement kind expands; supported actions and dialogs still work.

- [x] T009 [US2] Add accordion/type/action tests in ../zunera-frontend/src/components/transactions/__tests__/ExpandableHistoryItem.spec.js
- [x] T010 [US2] Implement compact accessible row/details in ../zunera-frontend/src/components/transactions/ExpandableHistoryItem.vue and wire events through ../zunera-frontend/src/components/transactions/TransactionHistoryList.vue
- [x] T011 [US2] Preserve existing detail, mutation, and dialog wiring in ../zunera-frontend/src/views/transactions/TransactionsListView.vue

## Phase 5: User Story 3 - Understand period results and filters (P3)

**Independent Test**: Accurate period-only summary appears when comparable; filters and empty states remain clear.

- [x] T012 [US3] Add summary/filter/empty tests in ../zunera-frontend/src/views/transactions/__tests__/TransactionsListView.spec.js and ../zunera-frontend/src/components/transactions/__tests__/TransactionFilterBar.spec.js
- [x] T013 [US3] Fetch existing dashboard period summary and suppress incomparable totals in ../zunera-frontend/src/views/transactions/TransactionsListView.vue and ../zunera-frontend/src/components/transactions/TransactionFinancialSummary.vue
- [x] T014 [US3] Refine filter toolbar, chips, and empty states in ../zunera-frontend/src/components/transactions/TransactionFilterBar.vue and ../zunera-frontend/src/components/transactions/TransactionHistoryList.vue
- [x] T015 [US3] Add localized labels in ../zunera-frontend/src/i18n/messages.js

## Phase 6: Polish

- [x] T016 Add month, grouping, accordion, filters, actions, and responsive journeys in ../zunera-frontend/e2e/transactions.spec.js; verify Budgets in ../zunera-frontend/e2e/budgets.spec.js
- [x] T017 Run frontend unit tests, lint, build, targeted Playwright and inspect light/dark/320px in ../zunera-frontend/
- [x] T018 Check diff, financial values, task completion, and report limitations in specs/013-transactions-activity-redesign/tasks.md

## Dependencies

T003–T005 block month UX. T006–T008 establish list. T009–T011 establish accordion. T012–T015 establish summary and filters. T016–T018 validate combined behavior.

## Implementation Strategy

Preserve contracts and actions throughout. Complete each story with tests before next story; no backend migration or package work.
