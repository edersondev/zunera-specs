# Tasks: Monthly Budgets

**Input**: Design documents from `/specs/009-monthly-budgets/`  
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [budgets-api.yaml](contracts/budgets-api.yaml), [quickstart.md](quickstart.md)

**Tests**: Required. Include Laravel feature/unit/performance and contract
coverage, then frontend service/store/component and critical Playwright coverage.

**Organization**: User-story phases are independently testable. Every story's
backend contract, authorization, validation, service, and tests complete before
its frontend work begins.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Different file and no unfinished-task dependency.
- **[US#]**: User story phase only.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm coordinated workspace and executable artifact baseline.

- [ ] T001 Verify branch `009-monthly-budgets` in `/home/ederson/workspace/zunera/zunera-specs/.git`, `../zunera-backend/.git`, and `../zunera-frontend/.git` before edits.
- [ ] T002 [P] Validate contract source `specs/009-monthly-budgets/contracts/budgets-api.yaml` with Redocly and retain it as backend/frontend acceptance source.
- [ ] T003 [P] Create shared Budget test fixture builders in `../zunera-backend/tests/Support/Budgets/BudgetFixtures.php` for owners, categories, monthly budgets, plans, and exact-centavo transactions.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish protected data model and reusable owner-scoped budget
boundaries. No user-story work begins until this phase passes.

- [ ] T004 Create monthly budget/category-plan schema, unique constraints, snapshots, owner/month lookup index, factories, and model relations in `../zunera-backend/database/migrations/*_create_monthly_budgets_table.php`, `../zunera-backend/database/migrations/*_create_budget_category_plans_table.php`, `../zunera-backend/database/factories/MonthlyBudgetFactory.php`, `../zunera-backend/database/factories/BudgetCategoryPlanFactory.php`, `../zunera-backend/app/Models/MonthlyBudget.php`, and `../zunera-backend/app/Models/BudgetCategoryPlan.php`.
- [ ] T005 [P] Add month/year input data object, exact-centavo plan/copy data objects, and budget lifecycle/status enums in `../zunera-backend/app/Data/Budgets/BudgetMonthData.php`, `../zunera-backend/app/Data/Budgets/CreateBudgetPlanData.php`, `../zunera-backend/app/Data/Budgets/UpdateBudgetPlanData.php`, `../zunera-backend/app/Data/Budgets/CopyBudgetData.php`, and `../zunera-backend/app/Enums/Budgets/BudgetStatus.php`.
- [ ] T006 Extend category classification guards to reject non-expense reclassification once any plan exists in `../zunera-backend/app/Services/Categories/CategoryService.php` and `../zunera-backend/tests/Feature/Categories/CreateAndUpdateCategoriesTest.php`.
- [ ] T007 Implement owner lookup, active available expense-category eligibility, archived-plan mutability checks, snapshot capture, and conflict exception mapping in `../zunera-backend/app/Services/Budgets/BudgetService.php` and `../zunera-backend/app/Exceptions/Budgets/BudgetStateException.php`.
- [ ] T008 Implement shared full-calendar-month boundaries and source-scoped effective-expense aggregate foundation, excluding actual-view resource mapping and expected/projected aggregates, in `../zunera-backend/app/Services/Budgets/BudgetCalculationService.php` and `../zunera-backend/app/Services/Budgets/BudgetMonthResolver.php`.
- [ ] T009 Add authenticated month-read/create and category-plan Form Request, API Resource, thin controller, protected-route shell, and common 404/409/422 response behavior, excluding copy endpoint/request/resource handling, in `../zunera-backend/app/Http/Requests/Budgets/`, `../zunera-backend/app/Http/Resources/Budgets/`, `../zunera-backend/app/Http/Controllers/Api/V1/BudgetController.php`, and `../zunera-backend/routes/api.php`.
- [ ] T010 Add foundational model/service/request/resource contract coverage in `../zunera-backend/tests/Unit/Budgets/BudgetMonthResolverTest.php`, `../zunera-backend/tests/Unit/Budgets/BudgetCalculationServiceTest.php`, and `../zunera-backend/tests/Feature/Budgets/BudgetFoundationTest.php`.
- [ ] T011 Run foundational migration and backend suite in `../zunera-backend/database/migrations/`, `../zunera-backend/tests/Unit/Budgets/`, and `../zunera-backend/tests/Feature/Budgets/` before starting user-story phases.

**Checkpoint**: Owner-scoped month read, data invariants, calculation boundary,
and protected API shell exist. User stories may now proceed.

---

## Phase 3: User Story 1 - Create and Manage Monthly Plan (Priority: P1) 🎯 MVP

**Goal**: User creates empty monthly plan, adds/edits/removes eligible expense
category plans without changing transactions or balances.

**Independent Test**: Signed-in user creates month, plans Food, edits amount,
removes Food, and confirms financial source records/balances remain unchanged.

### Backend (complete first)

- [ ] T012 [P] [US1] Add create/add/update/remove plan feature and contract tests, including owner isolation, one-plan-per-category, money bounds, and empty creation, in `../zunera-backend/tests/Feature/Budgets/CreateAndManageBudgetsTest.php`.
- [ ] T013 [P] [US1] Add no-money-movement and category eligibility/classification-lock tests in `../zunera-backend/tests/Unit/Budgets/BudgetServiceTest.php` and `../zunera-backend/tests/Feature/Budgets/BudgetFinancialIsolationTest.php`.
- [ ] T014 [US1] Implement create-month and plan add/update/remove operations with transaction-safe uniqueness conflicts in `../zunera-backend/app/Services/Budgets/BudgetService.php` and `../zunera-backend/app/Http/Controllers/Api/V1/BudgetController.php`.
- [ ] T015 [US1] Complete create/plan request validation and Budget resource serialization against `specs/009-monthly-budgets/contracts/budgets-api.yaml` in `../zunera-backend/app/Http/Requests/Budgets/` and `../zunera-backend/app/Http/Resources/Budgets/`.
- [ ] T016 [US1] Run focused create/manage backend tests and publish verified contract behavior from `../zunera-backend/tests/Feature/Budgets/CreateAndManageBudgetsTest.php` before frontend starts.

### Frontend (after T016)

- [ ] T017 [P] [US1] Add budget Axios methods/data unwrap/CSRF error mapping tests in `../zunera-frontend/src/services/budgetService.js` and `../zunera-frontend/src/services/__tests__/budgetService.spec.js`.
- [ ] T018 [P] [US1] Add budget setup-store month/create/plan mutation/loading/error tests in `../zunera-frontend/src/stores/budgets/budgetStore.js` and `../zunera-frontend/src/stores/budgets/__tests__/budgetStore.spec.js`.
- [ ] T019 [P] [US1] Add positive centavo form, server-field-error, and duplicate-submit tests in `../zunera-frontend/src/components/budgets/BudgetPlanFormDialog.vue` and `../zunera-frontend/src/components/budgets/__tests__/BudgetPlanFormDialog.spec.js`.
- [ ] T020 [US1] Implement budget service and Pinia actions for month fetch, empty creation, plan create/update/remove, and successful selected-month refresh in `../zunera-frontend/src/services/budgetService.js` and `../zunera-frontend/src/stores/budgets/budgetStore.js`.
- [ ] T021 [US1] Implement plan form and removal confirmation with `CurrencyAmountInput`, top-labelled Element Plus form, field errors, disabled submit, and validation reset on closure in `../zunera-frontend/src/components/budgets/BudgetPlanFormDialog.vue` and `../zunera-frontend/src/components/budgets/RemoveBudgetPlanDialog.vue`.
- [ ] T022 [US1] Add authenticated route, primary navigation item, pt-BR/en labels, and thin compose-only page for create/manage flow in `../zunera-frontend/src/router/index.js`, `../zunera-frontend/src/layouts/AppShell.vue`, `../zunera-frontend/src/i18n/messages.js`, and `../zunera-frontend/src/views/budgets/BudgetsView.vue`.
- [ ] T023 [US1] Add create/edit/remove no-money-movement journey with accessible selectors in `../zunera-frontend/e2e/budgets.spec.js`.
- [ ] T024 [US1] Run US1 unit and browser tests from `../zunera-frontend/src/**/budgets/` and `../zunera-frontend/e2e/budgets.spec.js` after backend contract verification.

**Checkpoint**: MVP plan management works independently without financial movement.

---

## Phase 4: User Story 2 - Monitor Monthly Spending (Priority: P1)

**Goal**: User sees trustworthy category/month planned, realized, available,
utilization, status, unbudgeted, and total expense values.

**Independent Test**: Mixed effective/pending/removed/income/transfer source data
produces correct derived values; transaction correction updates next budget read.

### Backend (complete first)

- [ ] T025 [P] [US2] Add aggregate unit cases for effective/pending/removed/income/transfer exclusions, full calendar months, exact status thresholds, negative availability, and no-plan Not-applicable in `../zunera-backend/tests/Unit/Budgets/BudgetCalculationServiceTest.php`.
- [ ] T026 [P] [US2] Add feature tests for month read/resource fields, budgeted/unbudgeted/total values, owner isolation, and transaction-driven restatement in `../zunera-backend/tests/Feature/Budgets/ViewBudgetSpendingTest.php`.
- [ ] T027 [US2] Complete calculation projection/resource mapping for category and monthly actual values, current category state plus preserved snapshots, status/excess text inputs, and normal `budget: null` read in `../zunera-backend/app/Services/Budgets/BudgetCalculationService.php` and `../zunera-backend/app/Http/Resources/Budgets/`.
- [ ] T028 [US2] Run calculation and spending feature suites in `../zunera-backend/tests/Unit/Budgets/BudgetCalculationServiceTest.php` and `../zunera-backend/tests/Feature/Budgets/ViewBudgetSpendingTest.php` before frontend monitor work.

### Frontend (after T028)

- [ ] T029 [P] [US2] Add BRL/month/percent/Not-applicable/excess/status formatter tests in `../zunera-frontend/src/utils/budgets/budgetFormatters.js` and `../zunera-frontend/src/utils/budgets/__tests__/budgetFormatters.spec.js`.
- [ ] T030 [P] [US2] Add summary and category-row tests for values, 80/100/>100 text status, unbudgeted totals, and no-color-only meaning in `../zunera-frontend/src/components/budgets/__tests__/BudgetSummary.spec.js` and `../zunera-frontend/src/components/budgets/__tests__/BudgetPlanRow.spec.js`.
- [ ] T031 [US2] Implement pure presentation formatters and summary/list/row components that render server-derived actual values, semantic accessible progress, and explicit excess/status in `../zunera-frontend/src/utils/budgets/budgetFormatters.js`, `../zunera-frontend/src/components/budgets/BudgetSummary.vue`, `../zunera-frontend/src/components/budgets/BudgetPlanList.vue`, and `../zunera-frontend/src/components/budgets/BudgetPlanRow.vue`.
- [ ] T032 [US2] Integrate loading, error/retry, no-plan/no-expense state, and summary/list presentation into `../zunera-frontend/src/views/budgets/BudgetsView.vue` and `../zunera-frontend/src/stores/budgets/budgetStore.js`.
- [ ] T033 [US2] Add realized-exclusion, exceeded/approaching, unbudgeted-total, and text-status browser coverage in `../zunera-frontend/e2e/budgets.spec.js`.
- [ ] T034 [US2] Run US2 unit and browser tests from `../zunera-frontend/src/**/budgets/` and `../zunera-frontend/e2e/budgets.spec.js` after backend calculation tests pass.

**Checkpoint**: Monitoring is independently trustworthy and source corrections restate it.

---

## Phase 5: User Story 3 - Review Months and Copy Plan (Priority: P2)

**Goal**: User navigates calendar months, reviews history, and safely copies plan
inputs to a different empty month.

**Independent Test**: User opens historical September, copies it to empty October,
changes October independently, and verifies conflicts/source archive preserve
destination unchanged.

### Backend (complete first)

- [ ] T035 [P] [US3] Add month-bound/year-rollover, copy source/destination ownership, same/occupied destination, and atomic archived-source conflict feature tests in `../zunera-backend/tests/Feature/Budgets/CopyMonthlyBudgetTest.php`.
- [ ] T036 [P] [US3] Add archived snapshot/read-only/restored-mutable and classification-lock historical tests in `../zunera-backend/tests/Feature/Budgets/ArchivedBudgetCategoryTest.php`.
- [ ] T037 [US3] Implement owner-scoped full-month resolution, atomic copy with fresh active-category snapshot, archived-read-only mutation guard, and conflict mapping in `../zunera-backend/app/Services/Budgets/BudgetMonthResolver.php`, `../zunera-backend/app/Services/Budgets/BudgetService.php`, and `../zunera-backend/app/Exceptions/Budgets/BudgetStateException.php`.
- [ ] T038 [US3] Add copy endpoint/request/resource contract handling and run US3 backend suites in `../zunera-backend/app/Http/Requests/Budgets/CopyBudgetRequest.php`, `../zunera-backend/app/Http/Controllers/Api/V1/BudgetController.php`, and `../zunera-backend/tests/Feature/Budgets/` before frontend history/copy work.

### Frontend (after T038)

- [ ] T039 [P] [US3] Add month navigator and copy-dialog component tests for previous/next controls, destination validation, and conflict feedback in `../zunera-frontend/src/components/budgets/__tests__/BudgetMonthNavigator.spec.js` and `../zunera-frontend/src/components/budgets/__tests__/CopyBudgetDialog.spec.js`.
- [ ] T040 [P] [US3] Add archived snapshot/read-only and no-budget copy-CTA view tests in `../zunera-frontend/src/views/budgets/__tests__/BudgetsView.spec.js`.
- [ ] T041 [US3] Implement route-local month navigation, no-budget copy action, historical archived label/read-only affordances, and copy dialog state in `../zunera-frontend/src/components/budgets/BudgetMonthNavigator.vue`, `../zunera-frontend/src/components/budgets/BudgetEmptyState.vue`, `../zunera-frontend/src/components/budgets/CopyBudgetDialog.vue`, and `../zunera-frontend/src/views/budgets/BudgetsView.vue`.
- [ ] T042 [US3] Add month navigation, independent copy, occupied destination, archived-source, and keyboard dialog browser coverage in `../zunera-frontend/e2e/budgets.spec.js`.
- [ ] T043 [US3] Run US3 unit and browser tests from `../zunera-frontend/src/**/budgets/` and `../zunera-frontend/e2e/budgets.spec.js` after backend copy tests pass.

**Checkpoint**: History and all-or-nothing copy work independently.

---

## Phase 6: User Story 4 - Anticipate Future Expenses (Priority: P3)

**Goal**: User distinguishes actual spending from eligible pending expense
projection without changing balances or historical budget meaning.

**Independent Test**: Current/future plan with pending Food transaction shows
expected/projected fields; past month and recurrence definition alone do not.

### Backend (complete first)

- [ ] T044 [P] [US4] Add expected/projected unit cases for pending current/future expenses, generated occurrences, no raw recurrence projection, past-month omission, and actual-value invariance in `../zunera-backend/tests/Unit/Budgets/BudgetCalculationServiceTest.php`.
- [ ] T045 [P] [US4] Add expected/projected feature/resource and no-balance-effect tests in `../zunera-backend/tests/Feature/Budgets/BudgetProjectionTest.php`.
- [ ] T046 [US4] Complete pending-only projected calculation and resource fields while preserving actual availability/utilization/status in `../zunera-backend/app/Services/Budgets/BudgetCalculationService.php` and `../zunera-backend/app/Http/Resources/Budgets/`.
- [ ] T047 [US4] Run US4 projection backend suites in `../zunera-backend/tests/Unit/Budgets/BudgetCalculationServiceTest.php` and `../zunera-backend/tests/Feature/Budgets/BudgetProjectionTest.php` before frontend projection work.

### Frontend (after T047)

- [ ] T048 [P] [US4] Add projection display tests for realized/expected/projected separation, projected excess, and past-month omission in `../zunera-frontend/src/components/budgets/__tests__/BudgetPlanRow.spec.js` and `../zunera-frontend/src/components/budgets/__tests__/BudgetSummary.spec.js`.
- [ ] T049 [US4] Render projection-only labels/values/states in `../zunera-frontend/src/components/budgets/BudgetSummary.vue`, `../zunera-frontend/src/components/budgets/BudgetPlanRow.vue`, and `../zunera-frontend/src/utils/budgets/budgetFormatters.js` without client financial arithmetic.
- [ ] T050 [US4] Add pending/generated-occurrence/past-month projection browser coverage in `../zunera-frontend/e2e/budgets.spec.js`.
- [ ] T051 [US4] Run US4 unit and browser tests from `../zunera-frontend/src/**/budgets/` and `../zunera-frontend/e2e/budgets.spec.js` after backend projection tests pass.

**Checkpoint**: Projection is clearly forward-looking and never becomes realized truth.

---

## Phase 7: Polish and Cross-Cutting Concerns

**Purpose**: Complete performance, accessibility, contract, quality, and
cross-repository verification without expanding scope.

- [ ] T052 [P] Add 10,000-movement selected-month performance fixture/test and add composite source-query index only if measured need is demonstrated in `../zunera-backend/tests/Performance/Budgets/BudgetPerformanceTest.php` and `../zunera-backend/database/migrations/*_add_budget_aggregate_index_to_transactions_table.php`.
- [ ] T053 [P] Validate Light/Dark/System, 320px, 200% zoom, reduced motion, focus-return, alerts, 44px targets, and no-color-only progress in `../zunera-frontend/e2e/budgets.spec.js` and `../zunera-frontend/src/components/budgets/`.
- [ ] T054 Validate final API contract and quickstart scenarios in `specs/009-monthly-budgets/contracts/budgets-api.yaml` and `specs/009-monthly-budgets/quickstart.md`.
- [ ] T055 Run final backend tests/Pint and frontend unit/Playwright/build commands documented in `specs/009-monthly-budgets/quickstart.md`.

---

## Dependencies and Execution Order

```text
Setup → Foundational
Foundational → US1 (MVP)
US1 → US2
US1 → US3
US2 + US3 → US4
US1 + US2 + US3 + US4 → Polish
```

- US1 provides persisted planning lifecycle and frontend entry flow.
- US2 and US3 can proceed in parallel after US1, provided each respects
  backend-before-frontend order.
- US4 depends on US2 calculation projection/resource presentation; it may start
  backend test design after US2 but frontend waits for T047.

## Parallel Opportunities

- Setup: T002 and T003 may run together after T001.
- Foundation: T005 and T006 may run in parallel after T004; T010 test files may
  be prepared after T005–T009 contracts settle.
- US1: T012/T013 backend tests run in parallel; after T016, T017/T018/T019 run
  in parallel.
- US2: T025/T026 backend tests run in parallel; after T028, T029/T030 run in
  parallel.
- US3: T035/T036 backend tests run in parallel; after T038, T039/T040 run in
  parallel.
- US4: T044/T045 backend tests run in parallel; after T047, T048 may proceed
  before T049.
- Polish: T052 and T053 may run in parallel; T054/T055 follow completed work.

## Implementation Strategy

### MVP First (US1)

1. Complete T001–T011.
2. Complete T012–T024.
3. Verify a user manages a monthly plan while financial records stay unchanged.

### Incremental Delivery

1. Add trustworthy actual monitoring (US2).
2. Add month history and safe copy (US3).
3. Add clearly labelled pending-only projection (US4).
4. Finish measured performance and cross-cutting accessibility (Polish).
