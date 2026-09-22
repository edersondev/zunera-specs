# Tasks: Transactions Month Navigator

**Input**: Design documents from `specs/011-transactions-month-navigator/`
**Paths**: All paths are relative to the `zunera-specs` repository root; `../zunera-backend` and `../zunera-frontend` are sibling repositories.
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/financial-history-period-totals.yaml`, `quickstart.md`

**Tests**: Backend feature tests, frontend unit tests, and the transactions Playwright journey are required by the constitution and plan.

**Organization**: Tasks are grouped by user story. Backend period totals (US2) complete and are verified before the frontend month scope and shared control (US1, US3) consume them.

## Phase 1: Setup

**Purpose**: Establish matching branches and the contract baseline.

- [X] T001 Verify branch `011-transactions-month-navigator` is checked out in `zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.
- [X] T002 [P] Amend the canonical endpoint contract with `meta.totals` and its period rule in `specs/005-account-transfers/contracts/financial-history-api.yaml` and the totals note in `specs/005-account-transfers/data-model.md`.
- [X] T003 [P] Record the additive response contract in `specs/011-transactions-month-navigator/contracts/financial-history-period-totals.yaml`.

---

## Phase 2: Foundational

**Purpose**: Backend period-scoped totals that every frontend story depends on.

- [X] T004 [US2] Extend the aggregate with an optional inclusive period in `../zunera-backend/app/Services/FinancialHistory/FinancialHistoryService.php`, keeping all-time behavior when no period is supplied.
- [X] T005 [US2] Pass the validated filter data to the aggregate in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialHistoryController.php` so list and totals share one filter object.
- [X] T006 [P] [US2] Add period totals coverage — in-range, out-of-range, first/last-day boundaries, zero-value month, pending/removed exclusion, and no-period parity — in `../zunera-backend/tests/Feature/FinancialHistory/ListFinancialHistoryTest.php`.
- [X] T007 [US2] Update the aggregate call sites for the new signature in `../zunera-backend/tests/Feature/FinancialHistory/TransferReportingExclusionTest.php` and any other caller found by search.
- [X] T008 [US2] Run `docker exec zunera-backend-app-1 php artisan test --filter=FinancialHistory` and `docker exec zunera-backend-app-1 vendor/bin/pint --format=agent` from `../zunera-backend`.

**Checkpoint**: A history request with `from`/`to` returns totals for that period; a request without a period is unchanged.

---

## Phase 3: User Story 1 — Browse transactions by month (Priority: P1) 🎯 MVP

**Goal**: The transactions filter row carries a month navigator beside the search and filter controls, and the list follows the selected month with the address kept in sync.

**Independent Test**: Open the transactions page, see the current month beside the search and filter buttons, and step to another month with the arrows while the list and address follow.

### Frontend for User Story 1

- [X] T009 [P] [US1] Add month utility coverage — `businessMonth`, `formatMonth`, `shiftMonth`, `monthBounds` (including leap February and December), and `monthFromDate` — in `../zunera-frontend/src/utils/common/__tests__/monthFormatters.spec.js`.
- [X] T010 [US1] Move the month helpers into `../zunera-frontend/src/utils/common/monthFormatters.js` and re-export the existing names from `../zunera-frontend/src/utils/budgets/budgetFormatters.js` so budget code keeps working.
- [X] T011 [P] [US1] Add localized `common.month.previous` and `common.month.next` labels and remove the budgets-only month labels in `../zunera-frontend/src/i18n/messages.js`.
- [X] T012 [US1] Create the shared navigator in `../zunera-frontend/src/components/common/MonthNavigator.vue` with `month`, `loading`, `change-month`, `aria-live` label, and `month-previous`/`month-label`/`month-next` test hooks, replacing `../zunera-frontend/src/components/budgets/BudgetMonthNavigator.vue`.
- [X] T013 [P] [US1] Add navigator unit coverage in `../zunera-frontend/src/components/common/__tests__/MonthNavigator.spec.js` and remove the superseded budgets navigator spec.
- [X] T014 [US1] Point the budgets view at the shared navigator in `../zunera-frontend/src/views/budgets/BudgetsView.vue`.
- [X] T015 [US1] Add a `context` slot to `../zunera-frontend/src/components/layout/PageHeader.vue`, place the `MonthNavigator` there in `../zunera-frontend/src/views/transactions/TransactionsListView.vue` between the title and the page actions, and keep the search row limited to the search input plus the Search and Filters buttons in `../zunera-frontend/src/components/transactions/TransactionFilterBar.vue` (which keeps a `month` prop only as the period-criterion baseline).
- [X] T016 [P] [US1] Update filter-bar coverage for the two-column row, the `change-month` event, and narrow-screen stacking in `../zunera-frontend/src/components/transactions/__tests__/TransactionFilterBar.spec.js`.
- [X] T017 [US1] Own the selected month in `../zunera-frontend/src/views/transactions/TransactionsListView.vue`: derive it from the address period or the business month, request the inclusive month range with the other criteria, keep search/type/status/account/category on month change, reset pagination, and sync the address.
- [X] T018 [P] [US1] Update view coverage for the default month scope, month-change period application, retained criteria, and address sync in `../zunera-frontend/src/views/transactions/__tests__/TransactionsListView.spec.js`.

**Checkpoint**: Month navigation works on the transactions page with the address restored on reload.

---

## Phase 4: User Story 2 — Month totals match the visible movements (Priority: P2)

**Goal**: The realized cards describe the displayed month.

**Independent Test**: Switch between two months that hold different movements and see the three cards follow the visible month.

### Frontend for User Story 2

- [X] T019 [US2] Confirm the transactions view passes the period with every history request so `meta.totals` and the list share one filter object in `../zunera-frontend/src/views/transactions/TransactionsListView.vue`.
- [X] T020 [P] [US2] Add summary coverage for month-scoped and zero-value totals in `../zunera-frontend/src/components/transactions/__tests__/TransactionFinancialSummary.spec.js`.
- [X] T021 [US2] Verify the cards never retain the previous month's figures while a month change is in flight in `../zunera-frontend/src/stores/transactions/transactionStore.js` and its store spec.

**Checkpoint**: Cards and list always describe the same month.

---

## Phase 5: User Story 3 — One month control across screens (Priority: P3)

**Goal**: Budgets keeps its behavior through the shared control.

**Independent Test**: On the budgets page, the navigator still shows the selected month, crosses year boundaries, and disables while loading.

- [X] T022 [US3] Verify budgets month behavior, wording, and accessibility are unchanged with the shared component in `../zunera-frontend/src/views/budgets/__tests__/BudgetsView.spec.js`.
- [X] T023 [P] [US3] Confirm the budgets end-to-end month journey still passes against the shared control in `../zunera-frontend/e2e/budgets.spec.js`.

---

## Phase 6: Polish and cross-cutting verification

- [X] T024 [US1] Handle the custom-range, criterion-chip, clear-filters, and empty-state distinction rules in `../zunera-frontend/src/components/transactions/TransactionFilterBar.vue` and `../zunera-frontend/src/views/transactions/TransactionsListView.vue`.
- [X] T025 [US1] Test the custom range override, period criterion visibility, criterion removal, clear-keeps-month, and unfiltered empty state in `../zunera-frontend/src/components/transactions/__tests__/TransactionFilterBar.spec.js` and `../zunera-frontend/src/views/transactions/__tests__/TransactionsListView.spec.js`.
- [X] T026 [US1] Update the transactions Playwright journey so its API mock honors `from`/`to` for both list and totals, then assert the navigator beside the search controls, month change across the address and cards, and keyboard use at 320px in `../zunera-frontend/e2e/transactions.spec.js`.
- [X] T027 Run `npm run test:unit -- --run` and `npm run lint` from `../zunera-frontend`.
- [X] T028 Run `npm run build` then `CI=1 npm run test:e2e -- e2e/transactions.spec.js e2e/budgets.spec.js` from `../zunera-frontend`.
- [X] T029 Record implementation evidence and any deviations in `specs/011-transactions-month-navigator/quickstart.md` and commit the three repositories on branch `011-transactions-month-navigator`.

## Dependencies and Execution Order

- Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 5 → Phase 6.
- T004–T008 (backend totals) MUST complete before T019–T021 verify cards against real period totals.
- T010–T014 (shared navigator) MUST complete before T015 wires it into the filter row.
- T015–T018 MUST complete before T024–T026 refine custom-range, clearing, and end-to-end behavior.

## Parallel Execution Examples

- T002 and T003 (contract documentation) run together.
- T009, T011, and T013 (utilities, i18n, navigator coverage) run together once T010 defines the helper names.
- T016, T018, and T020 (filter bar, view, summary coverage) run together after their implementation tasks land.

## Implementation Strategy

1. MVP: T001–T008 plus T009–T018 deliver month browsing with period-accurate totals.
2. Then T019–T023 lock card accuracy and budgets parity.
3. Finish with T024–T029 for criteria semantics, end-to-end evidence, and repository commits.

## Completion Notes

- Backend: `FinancialHistoryService::totals` accepts the shared filter object and
  applies the inclusive `from`/`to` period; `FinancialHistoryController` builds the
  filter data once for the list and the totals. `php artisan test
  --filter=FinancialHistory` passes (13 tests) and Pint reports `passed`.
- Frontend: the navigator moved to `src/components/common/MonthNavigator.vue` and
  budgets now consumes it. On the transactions page it renders in the page header
  between the title and the "Nova transação" actions through a new `context` slot, and
  the filter row holds only the search input plus the Search and Filters buttons; below
  640px the header stacks the title, navigator, and actions. Vitest passes 430 tests and
  `npm run lint` passes.
- Address sync deviation: the first load keeps the address unchanged and applies the
  current business month to the request; the period is written to the address as soon
  as the user changes the month, matching the budgets screen, which also never
  rewrites the address on load.
- Playwright: `e2e/transactions.spec.js` and `e2e/budgets.spec.js` pass 42 tests across
  chromium, firefox, and webkit. Two budgets assertions previously matched an
  ambiguous `progressbar` role after the budget-overview redesign and were narrowed to
  the named "Progresso do orçamento" bar so the budgets month journey can run.
- Pre-existing failures outside this feature: `e2e/transfers.spec.js` (2) and
  `e2e/credit-cards.spec.js` (4) fail in all three browsers on the untouched baseline
  as well, because those specs expect unsigned realized totals, a
  `credit-card-history-label` hook the app no longer renders, and a dialog that stays
  unstable; none of them are caused by this change.
