# Tasks: Financial Goals

**Input**: [spec.md](spec.md), [plan.md](plan.md), [research.md](research.md), [data-model.md](data-model.md), [API contract](contracts/financial-goals-api.yaml), and [quickstart.md](quickstart.md).\
**Branches**: `015-financial-goals` in this repository, `../zunera-backend`, and `../zunera-frontend`.\
**Tests**: Required by the specification and project constitution. Write each backend story's tests before its behavior, pass the complete backend gate at T045, then begin frontend tests and implementation.\
**Delivery**: Complete every backend contract, owner check, validation rule, service, and test before any frontend implementation starts. Do not add packages or alter existing financial ledgers to represent goal activity.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm the three repositories and the approved design before editing either application.

- [ ] T001 Verify all three repositories are on `015-financial-goals` and review `specs/015-financial-goals/plan.md` plus `specs/015-financial-goals/contracts/financial-goals-api.yaml` before application edits.
- [ ] T002 Read `docs/design/design-foundation.md`, `docs/design/app-shell.md`, `docs/design/navigation.md`, and `docs/design/components.md`; note existing conventions for the planned views in `../zunera-frontend/src/views/goals/GoalOverviewView.vue`.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Persist owned goals, append-only activity, and idempotent mutation claims without a second balance ledger.

- [ ] T003 Create goal identity, ownership, metadata, status, account reference/snapshot, centavos target, and indexes in `../zunera-backend/database/migrations/2026_09_25_000001_create_financial_goals_table.php`; prevent ordinary account deletion from cascading to goals.
- [ ] T004 [P] Create append-only, owner-scoped monetary/lifecycle event storage with time-of-event account snapshots and ordered history indexes in `../zunera-backend/database/migrations/2026_09_25_000002_create_financial_goal_activities_table.php`.
- [ ] T005 [P] Create owner/key unique mutation-claim storage for operation fingerprint and replay response in `../zunera-backend/database/migrations/2026_09_25_000003_create_financial_goal_mutation_requests_table.php`.
- [ ] T006 [P] Add `FinancialGoal` and its owned activity/account relationships in `../zunera-backend/app/Models/FinancialGoal.php` without a copied account-balance field.
- [ ] T007 [P] Add `FinancialGoalActivity` with immutable amount, event type, business date, and account-at-time accessors in `../zunera-backend/app/Models/FinancialGoalActivity.php`.
- [ ] T008 [P] Add mutation-claim model and owner/key uniqueness behavior in `../zunera-backend/app/Models/FinancialGoalMutationRequest.php`.
- [ ] T009 Add migration/model invariant tests for ownership, account noncascade, centavos, event ordering, and claim uniqueness in `../zunera-backend/tests/Feature/FinancialGoals/GoalFoundationTest.php`; run on SQLite and the existing MySQL development schema.

**Checkpoint**: Shared persistence exists; existing Financial Accounts, Transactions, Transfers, Budgets, and credit-card tables remain untouched.

---

## Phase 3: Backend User Story 1 — Create a Goal and See Progress (Priority: P1) (API MVP)

**Goal**: Create an owned active goal, optionally designate initial money, and show truthful progress without changing assets.

**Independent Test**: Create a R$ 30.000,00 goal with R$ 3.000,00 initially designated from a R$ 10.000,00 account; see 10% progress, R$ 27.000,00 remaining, and unchanged account/asset figures. Also create an unlinked zero-allocation goal.

### Backend tests first

- [ ] T010 [P] [US1] Write create/list/detail contract, ownership, malformed amount, whitespace-only and overlong name, target/date, initial-allocation, and replay tests against `specs/015-financial-goals/contracts/financial-goals-api.yaml` in `../zunera-backend/tests/Feature/FinancialGoals/GoalCreationContractTest.php`; assert that a R$ 10.000,00 account with R$ 3.000,00 initially designated still has R$ 10.000,00 actual balance, R$ 3.000,00 designated, and R$ 7.000,00 unallocated.
- [ ] T011 [P] [US1] Write projection tests for zero, fractional and over-100% progress, remaining/excess, and integer-centavo correctness in `../zunera-backend/tests/Unit/FinancialGoals/GoalProjectionTest.php`.

### Backend implementation and gate

- [ ] T012 [P] [US1] Define validated create input, trim and reject blank names while enforcing the 200-character input limit, and add shared positive BRL centavos parsing in `../zunera-backend/app/Http/Requests/FinancialGoals/CreateFinancialGoalRequest.php` and `../zunera-backend/app/Data/FinancialGoals/GoalInput.php`.
- [ ] T013 [US1] Implement account eligibility, linked capacity locking, active/completed designation sums, and actual/designated/unallocated centavos for initial allocations in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalCapacityService.php`.
- [ ] T014 [US1] Implement owner-scoped list/detail projections and goal-only account-backing/progress fields, including actual/designated/unallocated account amounts, in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalQueryService.php` and `../zunera-backend/app/Http/Resources/FinancialGoals/FinancialGoalResource.php`.
- [ ] T015 [US1] Implement atomic create plus created/optional initial-allocation events and exact idempotent replay/conflict in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalMutationService.php` and `../zunera-backend/app/Services/FinancialGoals/FinancialGoalIdempotencyService.php`.
- [ ] T016 [US1] Expose protected owner-scoped create/list/detail endpoints and stable 401/404/409/422 responses in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialGoalController.php` and `../zunera-backend/routes/api.php`.
- [ ] T017 [US1] Run `../zunera-backend/tests/Feature/FinancialGoals/GoalCreationContractTest.php` and `../zunera-backend/tests/Unit/FinancialGoals/GoalProjectionTest.php`; resolve failures and verify contract, authorization, validation, and no account/transaction/budget side effects before advancing to US2 backend work.

**Backend checkpoint**: US1 contract, authorization, validation, and tests pass.

---

## Phase 4: Backend User Story 2 — Allocate and Withdraw Designated Money (Priority: P1)

**Goal**: Designate and release money with an exact, separate goal-activity trail.

**Independent Test**: Allocate R$ 500,00 to R$ 5.000,00, withdraw R$ 200,00, retry each request, and see R$ 5.300,00 plus two new monetary events with unchanged account/cash-flow figures.

### Backend tests first

- [ ] T018 [P] [US2] Write allocation/withdrawal/activity contract tests for positive centavos, owner scope, underflow, same-key replay, changed-key conflict, and chronological account snapshots in `../zunera-backend/tests/Feature/FinancialGoals/GoalMoneyActionContractTest.php`.
- [ ] T019 [P] [US2] Write simultaneous withdrawal and shared-account allocation tests, including a MySQL-backed overlapping-request case, in `../zunera-backend/tests/Feature/FinancialGoals/GoalConcurrencyTest.php`.

### Backend implementation and gate

- [ ] T020 [P] [US2] Add validated positive-amount requests for both money actions in `../zunera-backend/app/Http/Requests/FinancialGoals/GoalMoneyActionRequest.php`.
- [ ] T021 [US2] Implement atomic active-goal allocation/withdrawal with goal/account lock order, capacity/underflow checks, dated activity, and idempotent responses in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalMutationService.php`.
- [ ] T022 [US2] Add paged owner-only activity reads with stable descending event order and account-at-time context in `../zunera-backend/app/Http/Resources/FinancialGoals/FinancialGoalActivityResource.php` and `../zunera-backend/app/Services/FinancialGoals/FinancialGoalQueryService.php`.
- [ ] T023 [US2] Expose protected allocation, withdrawal, and activity endpoints in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialGoalController.php` and `../zunera-backend/routes/api.php`.
- [ ] T024 [US2] Run `../zunera-backend/tests/Feature/FinancialGoals/GoalMoneyActionContractTest.php` and `../zunera-backend/tests/Feature/FinancialGoals/GoalConcurrencyTest.php` against SQLite and MySQL; verify exactly-once events, serial capacity, and unchanged financial ledgers before advancing to US4 backend work.

**Backend checkpoint**: US2 contract, authorization, validation, and tests pass.

---

## Phase 5: Backend User Story 4 — Understand Account Association and Shortfalls (Priority: P2)

**Goal**: Edit goal metadata/account links and expose account coverage honestly when multiple goals share money.

**Independent Test**: Link two goals to one account, spend from the account, see exact negative unallocated/shortfall values, then reassociate or unlink without a transfer or rewritten activity.

### Backend tests first

- [ ] T025 [P] [US4] Write PATCH contract/ownership tests for trimmed and whitespace-only names, target, date, description, account change/null, capacity rejection, and prior event snapshots in `../zunera-backend/tests/Feature/FinancialGoals/GoalUpdateContractTest.php`.
- [ ] T026 [P] [US4] Write account-capacity tests for multiple active goals, later real spending, inactive/unavailable account, negative balance, and blocked additions in `../zunera-backend/tests/Feature/FinancialGoals/GoalAccountCoverageTest.php`.

### Backend implementation and gate

- [ ] T027 [P] [US4] Add PATCH validation/DTO for partial metadata and nullable account, trimming names and rejecting blank or overlong input, credit cards, foreign/inactive accounts, past newly set dates, and invalid centavos in `../zunera-backend/app/Http/Requests/FinancialGoals/UpdateFinancialGoalRequest.php` and `../zunera-backend/app/Data/FinancialGoals/GoalUpdateInput.php`.
- [ ] T028 [US4] Implement active-only metadata and account edits, ordered old/new account locking, destination capacity check, historical account preservation, and account-change event in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalMutationService.php`.
- [ ] T029 [US4] Extend the US1 account-coverage projection with exact shortfall, inactive-or-unavailable labels, and null unknown balances in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalCapacityService.php` and `../zunera-backend/app/Http/Resources/FinancialGoals/FinancialGoalResource.php`.
- [ ] T030 [US4] Expose owner-scoped PATCH and account-context error shapes in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialGoalController.php` and `../zunera-backend/routes/api.php`.
- [ ] T031 [US4] Run `../zunera-backend/tests/Feature/FinancialGoals/GoalUpdateContractTest.php` and `../zunera-backend/tests/Feature/FinancialGoals/GoalAccountCoverageTest.php`; verify no transfer/balance mutation before advancing to US3 backend work.

**Backend checkpoint**: US4 contract, authorization, validation, and tests pass.

---

## Phase 6: Backend User Story 3 — Manage Goal Lifecycle (Priority: P2)

**Goal**: Explicitly complete, reopen, archive, and restore goals while retaining money and history rules.

**Independent Test**: Reach target, complete, reject mutations until reopen, withdraw all, archive, restore at zero, and confirm all activity remains; reject completion during linked shortfall or account unavailability.

### Backend tests first

- [ ] T032 [P] [US3] Write lifecycle endpoint contract tests for completion eligibility, blocked completed/archived money, status, and PATCH edits, full-withdrawal archive gate, restore, and owner scope in `../zunera-backend/tests/Feature/FinancialGoals/GoalLifecycleContractTest.php`.
- [ ] T033 [P] [US3] Write cases for linked shortfall/inactive/unavailable completion rejection, completion after valid unlinking or reassociation, unlinked unverified completion, completed allocations still counting as account designation, and unchanged completed status after a later shortfall/archive in `../zunera-backend/tests/Feature/FinancialGoals/GoalCompletionBackingTest.php`.

### Backend implementation and gate

- [ ] T034 [US3] Implement explicit complete/reopen/archive/restore transitions, event timestamps, completion capacity recheck, and immutable completed/archived actions in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalMutationService.php`.
- [ ] T035 [US3] Expose protected lifecycle actions and actionable conflict errors in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialGoalController.php` and `../zunera-backend/routes/api.php`.
- [ ] T036 [US3] Run `../zunera-backend/tests/Feature/FinancialGoals/GoalLifecycleContractTest.php` and `../zunera-backend/tests/Feature/FinancialGoals/GoalCompletionBackingTest.php`; verify history/status, completed-goal designation, and financial invariants before advancing to US5 backend work.

**Backend checkpoint**: US3 contract, authorization, validation, and tests pass.

---

## Phase 7: Backend User Story 5 — Plan for a Target Date (Priority: P2)

**Goal**: Show calendar timing and simple contribution guidance without creating scheduled money movement.

**Independent Test**: With R$ 12.000,00 remaining and 12 inclusive contribution months, show R$ 1.000,00/month; due-today, overdue, undated, and funded goals show no recommendation.

### Backend tests first

- [ ] T037 [P] [US5] Write calendar-month, timezone, centavo-ceiling, due/overdue, no-date, and reached-target tests in `../zunera-backend/tests/Unit/FinancialGoals/GoalContributionCalculatorTest.php`.
- [ ] T038 [P] [US5] Test date and suggestion fields in create/detail/PATCH responses and absence of recurring activity in `../zunera-backend/tests/Feature/FinancialGoals/GoalDateGuidanceContractTest.php`.

### Backend implementation and gate

- [ ] T039 [US5] Implement inclusive remaining-month guidance and due/overdue timing in `../zunera-backend/app/Services/FinancialGoals/GoalContributionCalculator.php` and surface it in `../zunera-backend/app/Http/Resources/FinancialGoals/FinancialGoalResource.php`.
- [ ] T040 [US5] Run `../zunera-backend/tests/Unit/FinancialGoals/GoalContributionCalculatorTest.php` and `../zunera-backend/tests/Feature/FinancialGoals/GoalDateGuidanceContractTest.php`; verify no recurring transaction creation before advancing to US6 backend work.

**Backend checkpoint**: US5 contract, authorization, validation, and tests pass.

---

## Phase 8: Backend User Story 6 — Review Goals Without Distorting Financial Views (Priority: P2)

**Goal**: Summarize active goals and show a compact Dashboard selection separate from cash flow.

**Independent Test**: Compare Dashboard, Accounts, Budgets, history, cards, and cash-flow values before/after goal actions; only goal-specific overview and Dashboard card values change.

### Backend tests first

- [ ] T041 [P] [US6] Write active-only summary, separate unverified subtotal, completed designation in account capacity, pagination, and three-goal ordering contract tests in `../zunera-backend/tests/Feature/FinancialGoals/GoalOverviewDashboardContractTest.php`.
- [ ] T042 [P] [US6] Write regression tests across account totals, transaction/transfer history, budget utilization, Dashboard realized/expected cash flow, credit-card purchases/payments, and recurring card recognition in `../zunera-backend/tests/Feature/FinancialGoals/GoalFinancialIntegrityTest.php`.

### Backend implementation and gate

- [ ] T043 [US6] Implement active-only target/allocated/per-goal remaining totals, unverified subtotal, attention counts, and bounded three-goal selection in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalQueryService.php`.
- [ ] T044 [US6] Expose protected `/financial-goals/summary` and `/financial-dashboard/goals` using `../zunera-backend/app/Http/Controllers/Api/V1/FinancialGoalDashboardController.php`, `../zunera-backend/app/Http/Resources/FinancialGoals/FinancialGoalSummaryResource.php`, and `../zunera-backend/routes/api.php`.
- [ ] T045 [US6] Run the complete backend test suite using `../zunera-backend/phpunit.xml` and run Pint; include all FinancialGoals contract/concurrency and existing Dashboard/Budget/CreditCard/Recurring tests, verify SQLite and MySQL migration/capacity behavior, then check the published API contract at `specs/015-financial-goals/contracts/financial-goals-api.yaml` before any frontend work.

**Backend checkpoint**: US6 contract, authorization, validation, and tests pass.

---

## Phase 9: Frontend Delivery (After Complete Backend Gate)

**Purpose**: Consume the verified Financial Goals API only after the full backend gate at T045.

### Frontend User Story 1 — Create a Goal and See Progress (Priority: P1)

**Prerequisite**: T045 complete; use the verified backend contract and the independent test in the matching backend story phase.

- [ ] T046 [P] [US1] Test create/list/detail request shapes, idempotency key reuse, and error mapping in `../zunera-frontend/src/services/__tests__/financialGoalService.spec.js`.
- [ ] T047 [US1] Implement goal API reads/create and shared owned-goal state in `../zunera-frontend/src/services/financialGoalService.js` and `../zunera-frontend/src/stores/goals/financialGoalStore.js`.
- [ ] T048 [P] [US1] Build accessible numeric/text progress with visual fill capped at 100% and explicit excess in `../zunera-frontend/src/components/goals/GoalProgress.vue` and its test `../zunera-frontend/src/components/goals/__tests__/GoalProgress.spec.js`.
- [ ] T049 [US1] Build name/target/date/account/notes/initial-amount form using existing currency input and server validation in `../zunera-frontend/src/components/goals/GoalFormDialog.vue` and `../zunera-frontend/src/views/goals/GoalOverviewView.vue`.
- [ ] T050 [US1] Build owned goal detail with initial progress, escaped name/notes, unverified-backing label, and empty states in `../zunera-frontend/src/views/goals/GoalDetailView.vue`; add protected overview/detail routes in `../zunera-frontend/src/router/index.js` and a basic Goals link in `../zunera-frontend/src/components/navigation/AppNavigation.vue` and `../zunera-frontend/src/layouts/AppShell.vue`.
- [ ] T051 [US1] Verify the in-app Goals link and create-to-detail journey, account balance invariance, unlinked label, safe rendering of hostile name/notes text, mobile readability, and PT-BR/English text in `../zunera-frontend/e2e/financial-goals.spec.js` and `../zunera-frontend/src/i18n/messages.js`.

**Checkpoint**: US1 works without allocation maintenance or lifecycle actions; it does not affect any financial balance.

### Frontend User Story 2 — Allocate and Withdraw Designated Money (Priority: P1)

**Prerequisite**: T045 complete; use the verified backend contract and the independent test in the matching backend story phase.

- [ ] T052 [P] [US2] Extend service/store tests for allocation, withdrawal, activity paging, retry-safe keys, and server refresh in `../zunera-frontend/src/stores/goals/__tests__/financialGoalStore.spec.js`.
- [ ] T053 [US2] Add money-action and activity methods to `../zunera-frontend/src/services/financialGoalService.js` and `../zunera-frontend/src/stores/goals/financialGoalStore.js`.
- [ ] T054 [P] [US2] Build amount dialog with explicit designate/release copy and validation in `../zunera-frontend/src/components/goals/GoalAmountDialog.vue`; build dated, account-aware event list in `../zunera-frontend/src/components/goals/GoalActivityList.vue`.
- [ ] T055 [US2] Wire actions/history into `../zunera-frontend/src/views/goals/GoalDetailView.vue` and verify allocate–withdraw–retry behavior in `../zunera-frontend/e2e/financial-goals.spec.js`.

**Checkpoint**: US1 and US2 independently pass their journeys; no goal event is a financial transaction.

### Frontend User Story 4 — Understand Account Association and Shortfalls (Priority: P2)

**Prerequisite**: T045 complete; use the verified backend contract and the independent test in the matching backend story phase.

- [ ] T056 [P] [US4] Test shortfall, unverified, inactive, actual/designated/unallocated text and accessible non-color cues in `../zunera-frontend/src/components/goals/__tests__/GoalAccountCoverage.spec.js`.
- [ ] T057 [US4] Add PATCH/edit flow and eligible account selection in `../zunera-frontend/src/services/financialGoalService.js`, `../zunera-frontend/src/stores/goals/financialGoalStore.js`, and `../zunera-frontend/src/components/goals/GoalFormDialog.vue`.
- [ ] T058 [US4] Show coverage/shortfall and corrective actions in `../zunera-frontend/src/components/goals/GoalAccountCoverage.vue` and `../zunera-frontend/src/views/goals/GoalDetailView.vue`; verify shared-account shortage and reassociation in `../zunera-frontend/e2e/financial-goals.spec.js`.

**Checkpoint**: Account balance stays authoritative; shortage is visible without altering goal history.

### Frontend User Story 3 — Manage Goal Lifecycle (Priority: P2)

**Prerequisite**: T045 complete; use the verified backend contract and the independent test in the matching backend story phase.

- [ ] T059 [P] [US3] Add store tests for available actions and server conflict responses across active/completed/archived states in `../zunera-frontend/src/stores/goals/__tests__/financialGoalLifecycle.spec.js`.
- [ ] T060 [US3] Add lifecycle methods and explicit reopen/full-withdrawal guidance in `../zunera-frontend/src/stores/goals/financialGoalStore.js` and `../zunera-frontend/src/views/goals/GoalDetailView.vue`.
- [ ] T061 [US3] Add completed/archived list access and retained activity presentation in `../zunera-frontend/src/views/goals/GoalOverviewView.vue`; verify complete–reopen–withdraw–archive–restore in `../zunera-frontend/e2e/financial-goals.spec.js`.

**Checkpoint**: Goal status changes are explicit and auditable; completed allocation still designates account money.

### Frontend User Story 5 — Plan for a Target Date (Priority: P2)

**Prerequisite**: T045 complete; use the verified backend contract and the independent test in the matching backend story phase.

- [ ] T062 [P] [US5] Test guidance wording, overdue state, and no-guarantee copy in `../zunera-frontend/src/components/goals/__tests__/GoalDateGuidance.spec.js`.
- [ ] T063 [US5] Render backend-provided date/time and suggested monthly centavos as guidance in `../zunera-frontend/src/components/goals/GoalDateGuidance.vue` and `../zunera-frontend/src/views/goals/GoalDetailView.vue`; verify date states in `../zunera-frontend/e2e/financial-goals.spec.js`.

**Checkpoint**: Guidance is a calculation only; no recurring rule or financial entry is created.

### Frontend User Story 6 — Review Goals Without Distorting Financial Views (Priority: P2)

**Prerequisite**: T045 complete; use the verified backend contract and the independent test in the matching backend story phase.

- [ ] T064 [P] [US6] Test overview totals/filters, loading and unavailable states without unknown-as-zero values, safe rendering of hostile goal names/notes, and independent Dashboard loading/error/retry behavior in `../zunera-frontend/src/views/goals/__tests__/GoalOverviewView.spec.js` and `../zunera-frontend/src/components/dashboard/__tests__/DashboardGoalsCard.spec.js`.
- [ ] T065 [US6] Render active totals, completed/archived sections, unverified subtotal, attention, empty, loading, and unavailable states without showing unknown values as zero; escape goal names/notes in `../zunera-frontend/src/views/goals/GoalOverviewView.vue` and `../zunera-frontend/src/components/goals/GoalCard.vue`.
- [ ] T066 [US6] Extend the basic Goals navigation with completed/archived history access and navigation-state tests in `../zunera-frontend/src/components/navigation/AppNavigation.vue`, `../zunera-frontend/src/layouts/AppShell.vue`, and `../zunera-frontend/src/router/index.js`.
- [ ] T067 [US6] Add independently fetched compact goals card without changing existing Dashboard financial selectors/totals in `../zunera-frontend/src/components/dashboard/DashboardGoalsCard.vue` and `../zunera-frontend/src/views/dashboard/FinancialDashboardView.vue`.
- [ ] T068 [US6] Complete PT-BR/English strings, visible status/shortfall wording, hostile-content rendering checks, and navigation/dashboard regression journeys in `../zunera-frontend/src/i18n/messages.js`, `../zunera-frontend/e2e/financial-goals.spec.js`, and `../zunera-frontend/e2e/financial-dashboard.spec.js`.

**Checkpoint**: Goals are findable and understandable; established financial figures keep their original meaning.

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Verify the whole feature against performance, accessibility, security, and financial-integrity gates.

- [ ] T069 [P] Run the complete backend suite via `../zunera-backend/phpunit.xml` and Pint, including `../zunera-backend/tests/Feature/FinancialGoals/GoalFinancialIntegrityTest.php`; fix failures without relaxing existing financial expectations.
- [ ] T070 [P] Run frontend Vitest, ESLint, build, and isolated Playwright goals/dashboard journeys from `../zunera-frontend/`, including `../zunera-frontend/e2e/financial-goals.spec.js`.
- [ ] T071 Exercise 100 owned goals/1,000 activities against overview/detail and inspect query counts and p95 usable-content timing against `specs/015-financial-goals/quickstart.md`; remove material N+1 or slow-path issues in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalQueryService.php`.
- [ ] T072 Verify 320 px/200% zoom, keyboard/screen-reader labels, light/dark/system themes, and non-color status cues against `docs/design/design-foundation.md` in `../zunera-frontend/src/views/goals/GoalOverviewView.vue` and `../zunera-frontend/src/views/goals/GoalDetailView.vue`.
- [ ] T073 Reconcile implemented request/response/error shapes and full acceptance walkthrough with `specs/015-financial-goals/contracts/financial-goals-api.yaml` and `specs/015-financial-goals/quickstart.md`; correct any drift in those artifacts or application files before delivery.
- [ ] T074 Run the uncoached SC-001 creation and SC-004 shortfall-understanding checks with at least 10 representative users; record per-person time, correctness, and both 90% pass rates in `specs/015-financial-goals/checklists/usability.md`, then resolve and repeat any failed criterion before delivery.

---

## Dependencies & Execution Order

### Phase dependencies

1. Setup (T001–T002) precedes Foundation (T003–T009); Foundation blocks all stories.
2. Finish every backend story before frontend work: US1 (T010–T017) → US2 (T018–T024) → US4 (T025–T031) → US3 (T032–T036) → US5 (T037–T040) → US6 (T041–T045). US4 precedes US3 so unlinking and reassociation exist before completion rules are delivered.
3. T045 is the feature-wide backend gate. The contract, ownership, validation, services, SQLite/MySQL checks, regression tests, and style checks must pass before any frontend task begins.
4. Frontend delivery (T046–T068) follows the same story dependency order and consumes the verified backend contract. Write frontend tests before implementation where practical.
5. Polish (T069–T074) follows backend and frontend delivery. A [P] task is parallel only when its prerequisites are satisfied and its files are distinct; shared services, routes, views, stores, and E2E files stay sequential.

```text
Setup → Foundation → Backend US1 → US2 → US4 → US3 → US5 → US6
                                                       ↓
                                            Full backend gate T045
                                                       ↓
                     Frontend US1 → US2 → US4 → US3 → US5 → US6 → Polish
```

### Contract and entity map

| Artifact | Owning backend story |
|---|---|
| Goal/activity/mutation persistence | Foundation |
| `GET/POST /financial-goals`, `GET /financial-goals/{goal_id}`; progress, initial allocation, and basic account coverage | US1 |
| `GET .../activities`, `POST .../allocations`, `POST .../withdrawals` | US2 |
| `PATCH /financial-goals/{goal_id}`; association and shortfall coverage | US4 |
| `POST .../complete`, `/reopen`, `/archive`, `/restore` | US3 |
| Target-date and monthly guidance fields on goal reads | US5 |
| `GET /financial-goals/summary`, `GET /financial-dashboard/goals` | US6 |

### Parallel examples

| Story | Safe parallel work after prerequisites |
|---|---|
| US1 | T010 contract tests with T011 projection tests; T046 service tests with T048 progress component after T045. |
| US2 | T018 money contract tests with T019 concurrency tests; T052 store tests with T054 isolated components after T045. |
| US4 | T025 PATCH tests with T026 account-coverage tests; T056 coverage component tests after T045. |
| US3 | T032 lifecycle contract tests with T033 backing tests; T059 store tests after T045. |
| US5 | T037 calculator tests with T038 response tests; T062 guidance tests after T045. |
| US6 | T041 summary/Dashboard tests with T042 financial regression tests; T064 view/card tests after T045. |

## Implementation Strategy

**Backend API MVP**: Finish Setup, Foundation, and backend US1 (T001–T017). Creation and detail already expose accurate progress plus actual, designated, and unallocated account money.\
**Frontend start**: Finish the remaining backend stories and pass T045 before T046. The US1 interface follows at T046–T051; later frontend stories add money actions, account edits, lifecycle, guidance, and aggregate discovery.\
**Final acceptance**: Use `quickstart.md` to prove cent-accurate amounts, owner isolation, exactly-once actions, real-account capacity, no financial double count, accessibility, the stated scale target, and both representative-user success criteria.
