# Tasks: Financial Dashboard

**Input**: Design documents from `specs/008-financial-dashboard/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/financial-dashboard-api.yaml`, `quickstart.md`

**Tests required**: Yes. Specification SQR-003 and Constitution require backend
unit/feature/performance, contract, frontend service/store/component, and
Playwright coverage.

## Phase 1: Setup

**Purpose**: Confirm feature workspace and make contract/test locations available.

- [x] T001 Verify branch `008-financial-dashboard` in `zunera-specs/.git`, `../zunera-backend/.git`, and `../zunera-frontend/.git`
- [x] T002 [P] Validate `specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml` with Redocly and retain it as API source of truth
- [x] T003 [P] Add Financial Dashboard fixture builders in `../zunera-backend/tests/Support/FinancialDashboard/FinancialDashboardFixtures.php`
- [x] T004 [P] Add dashboard API mock payload fixtures in `../zunera-frontend/src/services/__tests__/fixtures/dashboardFixtures.js`

---

## Phase 2: Backend Foundational

**Purpose**: Shared validated period and resource/controller boundary. Complete
before any user-story implementation.

- [x] T005 Create `DashboardPeriodData` preset/custom inclusive-date DTO in `../zunera-backend/app/Data/FinancialDashboard/DashboardPeriodData.php`
- [x] T006 Create authenticated custom-period validation in `../zunera-backend/app/Http/Requests/FinancialDashboard/DashboardPeriodRequest.php`
- [x] T007 Create dashboard API controller and protected route registration in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialDashboardController.php` and `../zunera-backend/routes/api.php`
- [x] T008 [P] Create shared money/period/account/category resource shapes in `../zunera-backend/app/Http/Resources/FinancialDashboard/`
- [x] T009 [P] Add `DashboardPeriodRequest` period-validation cases plus unauthenticated route-boundary cases in `../zunera-backend/tests/Feature/FinancialDashboard/FinancialDashboardFoundationTest.php`
- [x] T010 Run `php artisan test --filter=FinancialDashboard` and lint `specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml` after T005–T009

**Checkpoint**: Protected backend contract boundary exists; user stories can
start after T010. Frontend foundation (T016–T021) starts only after the US1
backend checkpoint T015, keeping backend implementation ahead of frontend work.

---

## Phase 3: User Story 1 — See Current Position and Period Result (Priority: P1) 🎯 MVP

**Goal**: Show current active-account balance plus selected-period realized income,
expenses, and result; transfers/planned state never affect realized totals.

**Independent Test**: User changes among current month, previous month, and valid
custom dates, then sees accurate centavo totals and current balance label.

### Backend — complete first

- [x] T011 [P] [US1] Write summary contract/calculation cases for effective, pending, removed, transfer, boundaries, and effective future-dated edits in `../zunera-backend/tests/Feature/FinancialDashboard/DashboardSummaryTest.php`
- [x] T012 [P] [US1] Write exact-centavo income/expense/result unit cases in `../zunera-backend/tests/Unit/FinancialDashboard/DashboardSummaryServiceTest.php`
- [x] T013 [US1] Implement owner/date/state-constrained realized summary and active current balance derivation in `../zunera-backend/app/Services/FinancialDashboard/DashboardSummaryService.php`
- [x] T014 [US1] Implement `GET /financial-dashboard/summary` controller action and summary API resource in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialDashboardController.php` and `../zunera-backend/app/Http/Resources/FinancialDashboard/DashboardSummaryResource.php`
- [x] T015 [US1] Run summary feature/unit tests and verify contract fields against `specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml`

### Frontend foundation — after T015

- [x] T016 Create six-slice dashboard Axios service in `../zunera-frontend/src/services/dashboardService.js`
- [x] T017 Create independent slice loading/error/retry Pinia state in `../zunera-frontend/src/stores/dashboard/dashboardStore.js`
- [x] T018 [P] Add BRL/date/movement/interval formatting helpers in `../zunera-frontend/src/utils/dashboard/dashboardFormatters.js`
- [x] T019 [P] Add service request/response tests in `../zunera-frontend/src/services/__tests__/dashboardService.spec.js`
- [x] T020 [P] Add independent-slice loading/error/retry tests in `../zunera-frontend/src/stores/dashboard/__tests__/dashboardStore.spec.js`
- [x] T021 Run `npm run test:unit -- --run src/services/__tests__/dashboardService.spec.js src/stores/dashboard/__tests__/dashboardStore.spec.js` after T016–T020

### Frontend UI — after T021

- [x] T022 [P] [US1] Build accessible preset/custom date selector with explicit props/emits in `../zunera-frontend/src/components/dashboard/DashboardPeriodSelector.vue`
- [x] T023 [P] [US1] Build current-balance and realized-result cards with non-color result cues in `../zunera-frontend/src/components/dashboard/FinancialSummaryCards.vue`
- [x] T024 [P] [US1] Add period-selector and summary-card state/accessibility tests in `../zunera-frontend/src/components/dashboard/__tests__/DashboardPeriodSelector.spec.js` and `../zunera-frontend/src/components/dashboard/__tests__/FinancialSummaryCards.spec.js`
- [x] T025 [US1] Compose dashboard landing view, current-month default, custom-period fetch, and current-balance distinction in `../zunera-frontend/src/views/dashboard/FinancialDashboardView.vue`
- [x] T026 [US1] Replace protected-home placeholder with dashboard landing and Dashboard labels in `../zunera-frontend/src/router/index.js`, `../zunera-frontend/src/layouts/AppShell.vue`, and `../zunera-frontend/src/i18n/messages.js`
- [x] T027 [US1] Add current-month, period-switch, custom-range, transfer-exclusion, and result-cue E2E coverage in `../zunera-frontend/e2e/financial-dashboard.spec.js`

**Checkpoint**: US1 backend is functional and independently testable after T015;
its frontend is complete after T021–T027.

---

## Phase 4: User Story 2 — Understand Spending and Change Over Time (Priority: P2)

**Goal**: Explain realized expense allocation and period evolution without
misleading comparisons or color-only charts.

**Independent Test**: User with multi-category effective expenses selects period
and identifies ranked category totals/shares plus correctly scoped evolution.

### Backend — complete first

- [x] T028 [P] [US2] Write distribution cases for realized-only amounts, ranking, shares, archived categories, and empty periods in `../zunera-backend/tests/Feature/FinancialDashboard/DashboardExpenseDistributionTest.php`
- [x] T029 [P] [US2] Write evolution interval and partial-boundary cases in `../zunera-backend/tests/Feature/FinancialDashboard/DashboardEvolutionTest.php`
- [x] T030 [P] [US2] Implement owner-scoped realized expense category aggregation in `../zunera-backend/app/Services/FinancialDashboard/DashboardExpenseDistributionService.php`
- [x] T031 [P] [US2] Implement daily/weekly/monthly bucket selection and partial-boundary derivation in `../zunera-backend/app/Services/FinancialDashboard/DashboardEvolutionService.php`
- [x] T032 [US2] Add distribution/evolution routes, controller actions, and resources in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialDashboardController.php` and `../zunera-backend/app/Http/Resources/FinancialDashboard/`
- [x] T033 [US2] Run distribution/evolution tests and confirm response schemas in `specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml`

### Frontend — after T033

- [x] T034 [P] [US2] Build expense visualization with ranked accessible table/legend, archived label, and semantic category colors in `../zunera-frontend/src/components/dashboard/ExpenseDistributionCard.vue`
- [x] T035 [P] [US2] Build evolution visualization with interval labels, partial marker, and text/table alternative in `../zunera-frontend/src/components/dashboard/FinancialEvolutionCard.vue`
- [x] T036 [P] [US2] Add distribution/evolution rendering, empty, theme, and non-color accessibility tests in `../zunera-frontend/src/components/dashboard/__tests__/ExpenseDistributionCard.spec.js` and `../zunera-frontend/src/components/dashboard/__tests__/FinancialEvolutionCard.spec.js`
- [x] T037 [US2] Wire distribution/evolution slices, per-section retry, and responsive layout into `../zunera-frontend/src/views/dashboard/FinancialDashboardView.vue`
- [x] T038 [US2] Add expense/evolution, archived-category, partial-interval, keyboard, 320px, and theme E2E coverage in `../zunera-frontend/e2e/financial-dashboard.spec.js`

**Checkpoint**: US2 works independently against its two verified endpoints after
T033 and T034–T038.

---

## Phase 5: User Story 3 — Review Accounts and Recent Activity (Priority: P2)

**Goal**: Explain active-account allocation and up to ten newest activity records,
including transfer and Pending labels, with full-history navigation.

**Independent Test**: User identifies active account balances/allocation and
up to ten newest mixed records, then reaches Financial History for older activity.

### Backend — complete first

- [x] T039 [P] [US3] Write active-only account allocation, signed/over-100 allocation, zero-total unavailable-allocation, and archived-account cases in `../zunera-backend/tests/Feature/FinancialDashboard/DashboardAccountsTest.php`
- [x] T040 [P] [US3] Write up-to-ten newest (fewer than ten, exactly ten, more than ten) income/expense/transfer/pending/recurrence, discriminated category/account shapes, no-limit-parameter, and removed-record exclusion cases in `../zunera-backend/tests/Feature/FinancialDashboard/DashboardRecentActivityTest.php`
- [x] T041 [P] [US3] Implement active account overview/allocation projection in `../zunera-backend/app/Services/FinancialDashboard/DashboardAccountsService.php`
- [x] T042 [P] [US3] Implement up-to-ten newest-first mixed activity projection without caller-controlled limit in `../zunera-backend/app/Services/FinancialDashboard/DashboardRecentActivityService.php`
- [x] T043 [US3] Add accounts/recent routes, controller actions, and normalized resources in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialDashboardController.php` and `../zunera-backend/app/Http/Resources/FinancialDashboard/`
- [x] T044 [US3] Run account/activity feature tests and confirm limit/default/shape in `specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml`

### Frontend — after T044

- [x] T045 [P] [US3] Build active account balance/allocation card and no-account CTA in `../zunera-frontend/src/components/dashboard/AccountsOverviewCard.vue`
- [x] T046 [P] [US3] Build active-account allocation card that handles unavailable/signed shares, plus mixed activity card with transfer/Pending/recurrence labels and full-history link in `../zunera-frontend/src/components/dashboard/AccountsOverviewCard.vue` and `../zunera-frontend/src/components/dashboard/RecentActivityCard.vue`
- [x] T047 [P] [US3] Add accounts/activity empty, unavailable/signed allocation, discriminated category/account shape, label, limit, and navigation tests in `../zunera-frontend/src/components/dashboard/__tests__/AccountsOverviewCard.spec.js` and `../zunera-frontend/src/components/dashboard/__tests__/RecentActivityCard.spec.js`
- [x] T048 [US3] Wire accounts/recent slices and retry states into `../zunera-frontend/src/views/dashboard/FinancialDashboardView.vue`
- [x] T049 [US3] Add active-account, archived exclusion, ten-item, transfer, pending, recurrence-source, and history-link E2E cases in `../zunera-frontend/e2e/financial-dashboard.spec.js`

**Checkpoint**: US3 works independently against its verified backend sections
after T044 and T045–T049.

---

## Phase 6: User Story 4 — Distinguish Expected Activity (Priority: P3)

**Goal**: Show future pending transactions and eligible recurring dates in next
30 days as Expected, with no realized or duplicate effect.

**Independent Test**: User sees dated expected movements; a generated pending
occurrence appears once; expected values never enter current/realized sections.

### Backend — complete first

- [x] T050 [P] [US4] Write next-30-day pending transaction, active/paused/ended rule, archive, calendar, and duplicate-suppression cases in `../zunera-backend/tests/Feature/FinancialDashboard/DashboardUpcomingActivityTest.php`
- [x] T051 [P] [US4] Write recurrence schedule-window and generated-occurrence precedence unit cases in `../zunera-backend/tests/Unit/FinancialDashboard/DashboardUpcomingActivityServiceTest.php`
- [x] T052 [US4] Extend reusable future schedule-date calculation without changing recurrence lifecycle behavior in `../zunera-backend/app/Services/RecurringTransactions/RecurringScheduleCalculator.php`
- [x] T053 [US4] Implement future-only pending/eligible-recurrence projection and duplicate suppression in `../zunera-backend/app/Services/FinancialDashboard/DashboardUpcomingActivityService.php`
- [x] T054 [US4] Add upcoming route, controller action, resource, and 30-day horizon metadata in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialDashboardController.php` and `../zunera-backend/app/Http/Resources/FinancialDashboard/UpcomingActivityResource.php`
- [x] T055 [US4] Run upcoming feature/unit tests and verify `expected` state contract in `specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml`

### Frontend — after T055

- [x] T056 [P] [US4] Build Expected-only upcoming card with 30-day label and empty state in `../zunera-frontend/src/components/dashboard/UpcomingActivityCard.vue`
- [x] T057 [P] [US4] Add Expected label, no-duplication, and empty-state tests in `../zunera-frontend/src/components/dashboard/__tests__/UpcomingActivityCard.spec.js`
- [x] T058 [US4] Wire upcoming slice, independent retry, and realized/excluded presentation into `../zunera-frontend/src/views/dashboard/FinancialDashboardView.vue`
- [x] T059 [US4] Add upcoming horizon, pending/recurrence, duplicate, paused/ended, and no-realized-effect E2E coverage in `../zunera-frontend/e2e/financial-dashboard.spec.js`

**Checkpoint**: All user stories are independently functional after T055 and
T056–T059.

---

## Phase 7: Polish and Cross-Cutting Concerns

**Purpose**: Verify financial consistency, per-section resilience, accessibility,
performance, and delivery evidence across all stories.

- [x] T060 [P] Add cross-projection correction, ownership, and 10,000-movement ≤2-second performance coverage in `../zunera-backend/tests/Feature/FinancialDashboard/FinancialDashboardPerformanceTest.php`
- [x] T061 [P] Add dashboard route/view orchestration tests for independently failed/retried sections in `../zunera-frontend/src/views/dashboard/__tests__/FinancialDashboardView.spec.js`
- [x] T062 Run complete backend financial-dashboard suite and Pint in `../zunera-backend/tests/Feature/FinancialDashboard/` and `../zunera-backend/tests/Unit/FinancialDashboard/`
- [x] T063 Run dashboard unit suite, Playwright journey, and production build in `../zunera-frontend/src/components/dashboard/`, `../zunera-frontend/e2e/financial-dashboard.spec.js`, and `../zunera-frontend/package.json`
- [x] T064 Re-run quickstart validation and Redocly contract lint using `specs/008-financial-dashboard/quickstart.md` and `specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml`

---

## Dependencies and Execution Order

```text
Phase 1 Setup
  → Phase 2 Backend Foundation
    → US1 backend (P1 MVP) → Frontend foundation (T016–T021)
    → US2/US3/US4 backend (P2/P3, parallel after T010)
      → Story frontends (T022+, T034+, T045+, T056+)
        → Phase 7 Polish
```

- All stories require T010 (backend foundation) and T021 (frontend foundation).
- Each story's frontend work requires its backend checkpoint (T015, T033, T044,
  or T055); never start frontend contract consumption earlier.
- US2, US3, and US4 can run in parallel after Foundation when separately staffed;
  sequential delivery remains US1 → US2/US3 → US4.

## Parallel Examples

### User Story 1

```text
T011 and T012: summary feature/unit tests before T013
T022 and T023: selector and summary card after T021
```

### User Story 2

```text
T028–T031: distinct distribution/evolution tests and services
T034–T036: distinct chart components and tests after T033
```

### User Story 3

```text
T039–T042: distinct account/activity tests and services
T045–T047: distinct account/activity components and tests after T044
```

### User Story 4

```text
T050 and T051: feature/unit lifecycle coverage
T056 and T057: upcoming component and tests after T055
```

## Implementation Strategy

### MVP first

1. Finish T001–T010; setup and backend foundation.
2. Finish US1 backend through T015; validate its contract and tests.
3. Finish T016–T021; frontend contract adapter, formatters, and state slices.
4. Finish T022–T027; validate `/app` summary and period selection independently.

### Incremental delivery

1. Add US2 charts after verified aggregation routes.
2. Add US3 allocation/recent history after verified bounded projections.
3. Add US4 expected future activity last, preserving strict realized separation.
4. Finish Phase 7 before handoff.

## Execution Notes

Implementation landed on branch `008-financial-dashboard` in `zunera-specs`,
`../zunera-backend`, and `../zunera-frontend`.

- **Backend**: the whole suite runs in the project PHP container
  (`docker exec zunera-backend-app-1 php artisan test`) — 258 passed
  (1,758 assertions), including the 45 financial-dashboard feature, unit, and
  performance cases. The 10,000-movement performance test completes all six
  projections in 0.44s, inside the 2-second target. `vendor/bin/pint --test`
  passes repo-wide and the OpenAPI contract lints clean with Redocly.
- **Frontend**: 274 unit tests pass (46 new dashboard cases across service,
  store, formatter, component, and view orchestration), ESLint and Prettier are
  clean, the production build succeeds, and
  `CI=1 npx playwright test e2e/financial-dashboard.spec.js` passes 18/18 across
  chromium, firefox, and webkit.
- Host-shell note: the local PHP CLI has no `pdo_sqlite`/`pdo_mysql`, so the
  backend suites must run through the container (as the quickstart docker
  setup already provides).
- Full-suite Playwright run in CI mode reports 18 passed / 23 failed on
  chromium: every dashboard test passes, and all 23 failures are pre-existing
  specs unrelated to this feature. They fall into two groups — specs asserting
  English copy while `src/i18n/locale.js` defaults to `pt-BR`, and older specs
  whose selectors/copy no longer match the current screens. No failing spec
  touches the dashboard route, its components, or any identifier renamed here
  (the i18n change adds 26 lines and removes none), so they were left untouched.
