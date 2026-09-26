# Tasks: Financial Reports

**Input**: Design documents in specs/016-financial-reports/  
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [API contract](contracts/financial-reports-api.yaml), [quickstart.md](quickstart.md)

**Tests**: Required by SQR-002 and the constitution. Write focused failing tests before each behavior where practical; complete backend contract/authorization/validation/tests before any frontend task.

**Organization**: One backend overview and detail contract supplies all six stories. Phase 2 implements and verifies that indivisible backend foundation, including the paid-refund correction. Story phases then deliver independently testable frontend journeys in priority order. This satisfies the project-wide backend-first gate; no frontend task may start before T025 passes.

**Branch coordination**: Verify 016-financial-reports in specs, backend, and frontend before editing either app. Paths below are relative to the specs repository.

## Phase 1: Setup

**Purpose**: Confirm working context and prepare reusable financial fixtures.

- [ ] T001 Verify 016-financial-reports branches in specs, ../zunera-backend, and ../zunera-frontend; review specs/016-financial-reports/plan.md and specs/016-financial-reports/contracts/financial-reports-api.yaml before application edits.
- [ ] T002 Create owned fixture builders for ordinary transactions, statement installments, paid refunds, transfers, goals, and recurring occurrences in ../zunera-backend/tests/Support/FinancialReports/FinancialReportFixtures.php.

**Checkpoint**: Existing application branches and contract context verified; no packages or report-persistence tables added.

---

## Phase 2: Foundational Backend Gate

**Purpose**: Establish one authoritative, owner-scoped Reports contract and prove all financial semantics before frontend work.

**CRITICAL**: Complete T003–T025 before T026 or any other frontend task. The overview and contribution reads serve all stories, so backend work is grouped here rather than duplicated across story phases.

### Shared card-recognition prerequisite

- [ ] T003 [P] Write failing fully-paid, partly-paid, unpaid, multi-installment, cancellation, and replay refund-recognition cases in ../zunera-backend/tests/Feature/CreditCards/CreditEventRecognitionTest.php.
- [ ] T004 [P] Write failing Dashboard, Budget, and Financial History paid-refund parity cases in ../zunera-backend/tests/Feature/FinancialReports/ReportRecognitionParityTest.php.
- [ ] T005 Implement purchase-sequence allocation of accepted credit events to recognized installment expense, independent of statement payment state, in ../zunera-backend/app/Services/CreditCards/RecognizedCardExpenseProjection.php; preserve existing obligation and card-credit settlement effects.
- [ ] T006 [P] Change ../zunera-backend/app/Services/FinancialDashboard/RecurringCardExpenseProjection.php to use the shared recognized net expense from T005 for summary, evolution, and distribution.
- [ ] T007 [P] Change ../zunera-backend/app/Services/CreditCards/CreditCardBudgetProjectionService.php to use the shared recognized net expense from T005 without changing budget planning rules.
- [ ] T008 Update card entries in ../zunera-backend/app/Services/FinancialHistory/FinancialHistoryService.php to display the same recognized net amount and retain source credit-event traceability; make T003–T004 pass.
- [ ] T009 Run the focused card, Dashboard, Budget, and Financial History regressions from specs/016-financial-reports/quickstart.md and resolve all failures in ../zunera-backend/app/Services/CreditCards/RecognizedCardExpenseProjection.php.

### Scope, projections, and protected contract

- [ ] T010 [P] Write failing quick-choice, completed historical-month (`month=YYYY-MM`), custom, Sao Paulo date, leap/short-month, full-prior-month versus equal-day custom comparison, and percentage-eligibility tests in ../zunera-backend/tests/Unit/FinancialReports/ReportPeriodResolverTest.php.
- [ ] T011 Implement immutable period/filter scope and comparison resolver in ../zunera-backend/app/Data/FinancialReports/ReportScope.php and ../zunera-backend/app/Services/FinancialReports/ReportPeriodResolver.php.
- [ ] T012 [P] Write failing authentication, foreign/archived account and category, malformed range, missing/malformed/current-or-future historical `month`, incompatible preset/date/month parameters, incompatible type/category, and unknown-filter tests in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportAuthorizationTest.php.
- [ ] T013 Implement owner-scoped validation for overview and detail query parameters in ../zunera-backend/app/Http/Requests/FinancialReports/ReportScopeRequest.php and ../zunera-backend/app/Http/Requests/FinancialReports/ReportContributionRequest.php.
- [ ] T014 [P] Write failing exact-centavo ordinary/card/credit-event contribution and exclusion tests in ../zunera-backend/tests/Unit/FinancialReports/RecognizedContributionRepositoryTest.php.
- [ ] T015 Implement one owner-scoped ordinary/card/adjustment read model, shared filters, and archived-label resolution in ../zunera-backend/app/Repositories/FinancialReports/RecognizedContributionRepository.php.
- [ ] T016 [P] Write failing account income/direct expense/net, transfer direction, card settlement, pending exclusion, and type/category suppression cases in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportAccountActivityTest.php.
- [ ] T017 Implement account movement reads and account-filter attribution in ../zunera-backend/app/Repositories/FinancialReports/AccountMovementRepository.php.
- [ ] T018 [P] Write failing overview contract and reconciliation tests for summary, intervals, income/expense categories, accounts, comparison, empty/partial-section states, recurrence, goals, and card payment exclusion in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportOverviewContractTest.php.
- [ ] T019 Implement owner-scoped overview aggregation, shares, account-unattributed card amount, comparison, and explicit section availability in ../zunera-backend/app/Services/FinancialReports/FinancialReportService.php.
- [ ] T020 Expose validated overview through thin controller, Resource, and protected route in ../zunera-backend/app/Http/Controllers/Api/V1/FinancialReportController.php, ../zunera-backend/app/Http/Resources/FinancialReports/FinancialReportResource.php, and ../zunera-backend/routes/api.php.
- [ ] T021 [P] Write failing contract tests for metric IDs, current/prior scope, signed result/net contributions, paid-refund source links, cursor pages, all-record total, and foreign targets in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportContributionsContractTest.php.
- [ ] T022 Implement stable bounded contribution paging and all-record metric reconciliation over the shared read models in ../zunera-backend/app/Services/FinancialReports/FinancialReportContributionService.php.
- [ ] T023 Expose protected contribution detail and source links through ../zunera-backend/app/Http/Controllers/Api/V1/FinancialReportController.php, ../zunera-backend/app/Http/Resources/FinancialReports/FinancialReportContributionResource.php, and ../zunera-backend/routes/api.php.
- [ ] T024 Add 10,000-record/100-category/50-account contract and query-plan fixture checks in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportPerformanceTest.php; add a source-table index migration only if measured plans need it.
- [ ] T025 Complete backend gate: run focused and full Laravel tests plus Pint, validate response shapes against specs/016-financial-reports/contracts/financial-reports-api.yaml, and verify MySQL paid-refund and paged reconciliation using specs/016-financial-reports/quickstart.md.

**Checkpoint**: T003–T025 pass; backend API contract, authorization, validation, recognition repair, account movement, comparison, and drill-down are complete. Frontend may now start.

---

## Phase 3: User Story 1 - Understand a Period's Result (Priority: P1) 🎯 MVP

**Goal**: User opens Reports and understands realized income, expenses, and signed result for current month.

**Independent Test**: With effective income/expense fixtures, open protected Reports and reconcile three summary figures; only-income, only-expense, transfer, goal, and pending cases stay correct.

### Tests for User Story 1

- [ ] T026 [P] [US1] Write overview transport, applied-scope shape (including nullable historical month), and error-contract tests in ../zunera-frontend/src/services/__tests__/reportsService.spec.js.
- [ ] T027 [P] [US1] Write initial canonical URL scope normalization/restoration and stale-response, loading, and retry tests in ../zunera-frontend/src/composables/reports/__tests__/useReportScope.spec.js and ../zunera-frontend/src/stores/reports/__tests__/reportsStore.spec.js.
- [ ] T028 [P] [US1] Write signed-result, zero-side, and non-color summary tests in ../zunera-frontend/src/components/reports/__tests__/ReportSummary.spec.js.

### Frontend implementation (after T025)

- [ ] T029 [US1] Implement overview transport in ../zunera-frontend/src/services/reportsService.js, canonical URL query normalization/restoration for all contract scope fields and stale-request guards in ../zunera-frontend/src/composables/reports/useReportScope.js, and scoped read state in ../zunera-frontend/src/stores/reports/reportsStore.js. This shared URL foundation precedes US3 detail and US5 filters.
- [ ] T030 [US1] Add protected Reports route, primary navigation item, and PT-BR/English labels in ../zunera-frontend/src/router/index.js, ../zunera-frontend/src/layouts/AppShell.vue, and ../zunera-frontend/src/i18n/messages.js.
- [ ] T031 [US1] Compose thin current-month Reports view and realized summary in ../zunera-frontend/src/views/reports/ReportsView.vue and ../zunera-frontend/src/components/reports/ReportSummary.vue using existing currency/result formatters.
- [ ] T032 [US1] Add isolated current-month summary, income-only, expense-only, transfer/goal/pending exclusion, and unauthorized browser journeys in ../zunera-frontend/e2e/financial-reports.spec.js.

**Checkpoint**: Summary journey works independently against verified backend. This is minimum useful Reports release.

---

## Phase 4: User Story 2 - Explore Evolution and Categories (Priority: P1)

**Goal**: Explain summary through time intervals and separate expense/income categories, including recognized card adjustments.

**Independent Test**: Render short/long interval and category fixtures; interval/category sums match summary and post-payment refund appears in original expense category.

### Tests for User Story 2

- [ ] T033 [P] [US2] Write interval labels, partial-boundary, numeric-alternative, and income/expense distinction tests in ../zunera-frontend/src/components/reports/__tests__/ReportEvolution.spec.js.
- [ ] T034 [P] [US2] Write separate income/expense category, ranked share, archived category, and refund amount tests in ../zunera-frontend/src/components/reports/__tests__/ReportCategoryBreakdown.spec.js.

### Frontend implementation (after T025)

- [ ] T035 [P] [US2] Build readable interval visualization plus visible values using ../zunera-frontend/src/components/reports/ReportEvolution.vue and existing ../zunera-frontend/src/components/charts/BaseChart.vue.
- [ ] T036 [P] [US2] Build reusable income/expense ranked category analysis in ../zunera-frontend/src/components/reports/ReportCategoryBreakdown.vue; keep zero-denominator share unavailable.
- [ ] T037 [US2] Add both sections and correct empty states to ../zunera-frontend/src/views/reports/ReportsView.vue, reusing ../zunera-frontend/src/utils/dashboard/dashboardFormatters.js.
- [ ] T038 [US2] Add trend, category, recurring-card, and paid-refund browser checks to ../zunera-frontend/e2e/financial-reports.spec.js.

**Checkpoint**: Trends and both category distributions independently explain the verified summary.

---

## Phase 5: User Story 3 - Trace a Reported Amount (Priority: P1)

**Goal**: Every exposed category/summary amount can reveal paged signed source contributions without losing scope.

**Independent Test**: Open Food detail with ordinary expense and card refund; full signed sum equals displayed category total after all pages.

### Tests for User Story 3

- [ ] T039 [P] [US3] Write detail transport, metric/period/filter forwarding, and cursor error tests in ../zunera-frontend/src/services/__tests__/reportsContributionService.spec.js.
- [ ] T040 [P] [US3] Write signed card-adjustment, original-source identity, paging, all-record total, and keyboard-close tests in ../zunera-frontend/src/components/reports/__tests__/ReportContributionDrawer.spec.js.

### Frontend implementation (after T025)

- [ ] T041 [US3] Add contribution read and stale-page guards to ../zunera-frontend/src/services/reportsService.js and ../zunera-frontend/src/stores/reports/reportsStore.js.
- [ ] T042 [US3] Build paged report-specific source drawer in ../zunera-frontend/src/components/reports/ReportContributionDrawer.vue; show recognized date, signed metric contribution, refund source, and all-record total.
- [ ] T043 [US3] Wire summary/category detail events using the canonical URL scope established in T029 in ../zunera-frontend/src/views/reports/ReportsView.vue and ../zunera-frontend/src/components/reports/ReportCategoryBreakdown.vue.
- [ ] T044 [US3] Add summary/category drill-down, paid-refund trace, and multi-page centavo reconciliation journeys to ../zunera-frontend/e2e/financial-reports.spec.js.

**Checkpoint**: Category amount is traceable without relying on Transactions card-row behavior.

---

## Phase 6: User Story 4 - Compare Equivalent Periods (Priority: P2)

**Goal**: Show current/prior realized measures and neutral category differences with exact dates and safe percentages.

**Independent Test**: Compare completed months and March 1–31 versus February 1–28; signed differences remain correct and unsafe percentages are unavailable.

### Tests for User Story 4

- [ ] T045 [US4] Write prior/current date, zero/negative/sign-crossing, unequal-duration, category-change, and neutral-language tests in ../zunera-frontend/src/components/reports/__tests__/ReportComparison.spec.js.

### Frontend implementation (after T025)

- [ ] T046 [US4] Render server-derived comparison values and percentage-unavailable reasons in ../zunera-frontend/src/components/reports/ReportComparison.vue without client monetary recalculation.
- [ ] T047 [US4] Integrate comparison and current/prior metric drill-down in ../zunera-frontend/src/views/reports/ReportsView.vue and ../zunera-frontend/src/components/reports/ReportContributionDrawer.vue.
- [ ] T048 [US4] Add completed/partial period, direct-URL historical-month versus full-month custom comparison fixtures, previous-zero, negative-result, and category comparison browser cases to ../zunera-frontend/e2e/financial-reports.spec.js; period controls arrive in US6.

**Checkpoint**: Comparison is testable with existing default period and direct fixture scopes before expanded period controls.

---

## Phase 7: User Story 5 - Analyze Accounts and Filter Scope (Priority: P2)

**Goal**: Explain account-attributed flows separately from transfers/settlements and apply account/category/type filters consistently.

**Independent Test**: Account A income/direct expense/net stay distinct from outgoing transfer and card settlement; filters update every section/detail and clearly mark scope.

### Tests for User Story 5

- [ ] T049 [P] [US5] Write account direct-flow, transfer direction, settlement, unattributed card expense, and archived-account tests in ../zunera-frontend/src/components/reports/__tests__/ReportAccountActivity.spec.js.
- [ ] T050 [P] [US5] Write account/category/type chip, incompatible scope, reset, and movement-suppression tests in ../zunera-frontend/src/components/reports/__tests__/ReportFilterBar.spec.js.

### Frontend implementation (after T025)

- [ ] T051 [P] [US5] Build account activity list with distinct direct flow, transfer, settlement, and unattributed-card explanation in ../zunera-frontend/src/components/reports/ReportAccountActivity.vue.
- [ ] T052 [P] [US5] Build owned account/category/type controls and active filter chips with Element Plus in ../zunera-frontend/src/components/reports/ReportFilterBar.vue.
- [ ] T053 [US5] Wire filters through the canonical route query established in T029 to store scope, comparison, all sections, and contribution detail in ../zunera-frontend/src/views/reports/ReportsView.vue and ../zunera-frontend/src/stores/reports/reportsStore.js.
- [ ] T054 [US5] Add account metric drill-down and type/category suppression of transfer/settlement detail in ../zunera-frontend/src/components/reports/ReportContributionDrawer.vue.
- [ ] T055 [US5] Add account, category, type, combined-filter, reset, archived identity, and foreign-ID rejection journeys to ../zunera-frontend/e2e/financial-reports.spec.js.

**Checkpoint**: Filtered views never masquerade as workspace totals; account movement retains separate meaning.

---

## Phase 8: User Story 6 - Navigate Periods and Empty States (Priority: P2)

**Goal**: Offer five quick period choices, completed historical-month navigation, consistent custom boundaries, and clear empty/error states at scale.

**Independent Test**: Select each preset/custom range; every section/detail uses same returned dates, and no-activity/no-income/no-expense/no-prior/no-filter-match states remain distinct.

### Tests for User Story 6

- [ ] T056 [P] [US6] Write quick-choice/custom/completed historical-month navigation, inclusive boundary, and invalid-date tests in ../zunera-frontend/src/components/reports/__tests__/ReportPeriodSelector.spec.js.
- [ ] T057 [P] [US6] Extend the T027 URL scope tests with historical-month/custom restore, rapid period/filter changes, and stale-response cancellation in ../zunera-frontend/src/composables/reports/__tests__/useReportScope.spec.js.
- [ ] T058 [P] [US6] Write available-empty versus unavailable-section, retry, and filtered-empty tests in ../zunera-frontend/src/views/reports/__tests__/ReportsView.spec.js.

### Frontend implementation (after T025)

- [ ] T059 [P] [US6] Build five quick period choices, custom date selection, and completed historical-month navigation in ../zunera-frontend/src/components/reports/ReportPeriodSelector.vue.
- [ ] T060 [P] [US6] Extend the T029 URL scope composable with selector-to-route transitions for custom and historical-month choices, returning to current month, and rapid-change cleanup in ../zunera-frontend/src/composables/reports/useReportScope.js.
- [ ] T061 [US6] Integrate selector, resolved date labels, scope reset, section loading/empty/error/retry, and large-list readability in ../zunera-frontend/src/views/reports/ReportsView.vue.
- [ ] T062 [US6] Add current/previous month/year, navigated historical month, inclusive custom date, rapid changes, empty states, and unavailable-section browser journeys to ../zunera-frontend/e2e/financial-reports.spec.js.

**Checkpoint**: Every Reports section and detail reflects one visible period/filter context, including error and empty states.

---

## Phase 9: Polish & Cross-Cutting Verification

**Purpose**: Close required quality, scale, localization, and outcome evidence after all six stories.

- [ ] T063 [P] Verify PT-BR/English money/date/percentage copy, 320 px and 200% zoom, keyboard/assistive-technology access, light/dark/system themes, and chart text alternatives in ../zunera-frontend/e2e/financial-reports.spec.js.
- [ ] T064 [P] Record backend 10,000-record/100-category/50-account query results and frontend navigation timing against SC-005 in specs/016-financial-reports/checklists/performance.md using ../zunera-backend/tests/Feature/FinancialReports/FinancialReportPerformanceTest.php.
- [ ] T065 Run frontend unit suite, lint, build, and isolated Reports Playwright journeys from specs/016-financial-reports/quickstart.md; inspect auto-fix lint changes under ../zunera-frontend/src/ before completion.
- [ ] T066 Record at least 10 uncoached participant outcomes for SC-002/SC-003 without credentials or private transaction contents in specs/016-financial-reports/checklists/usability.md.
- [ ] T067 Reconcile acceptance fixtures, exact contract shapes, branch alignment, and every completed task against specs/016-financial-reports/spec.md, specs/016-financial-reports/contracts/financial-reports-api.yaml, and specs/016-financial-reports/tasks.md.

**Checkpoint**: Full feature validated; no report-only accounting model or unverified backend/frontend boundary remains.

---

## Dependencies & Execution Order

### Phase dependencies

1. Phase 1 precedes Phase 2.
2. T003–T009 repair recognized card spending before Reports aggregation. T010–T024 build and test complete backend report reads. T025 is the mandatory backend gate.
3. No frontend task T026–T063 begins before T025. Story phases proceed US1 → US2 → US3 → US4 → US5 → US6 for an integrated UI; each component/contract can be tested with fixtures independently.
4. Phase 9 follows the chosen story scope; T064 performance evidence can begin once backend performance fixture exists, but final outcome measurement follows the complete UI.

### Backend coverage mapped to stories

| Story | Foundational backend coverage |
|---|---|
| US1 | T003–T005, T010–T015, T018–T020, T025 |
| US2 | T003–T009, T014–T015, T018–T020, T025 |
| US3 | T014–T015, T021–T023, T025 |
| US4 | T010–T011, T018–T020, T025 |
| US5 | T012–T013, T016–T020, T021–T023, T025 |
| US6 | T010–T013, T018–T020, T024–T025 |

### User-story dependency graph

    Setup → Shared backend foundation/gate → US1
                                      US1 → US2 → US3
                                      US1 → US4
                                      US1 → US5
                                      US1 → US6
                            US2 + US3 + US4 + US5 + US6 → Polish

US2–US6 components and tests may be developed independently after T025; their browser integration uses the US1 route shell. US3 category click integration follows US2; US5 later adds account click integration. Backend payment/refund recognition and report reads are already complete before any story UI.

### Parallel opportunities

- T003 and T004 are independent failing regression test files. T006 and T007 can proceed in separate source files after T005.
- T010, T012, T014, T016, T018, and T021 are separate backend test files; write them independently, then implement shared services in dependency order.
- T026–T028, T033–T034, T039–T040, T049–T050, and T056–T058 cover separate frontend test files after the backend gate.
- T035/T036, T051/T052, and T059/T060 modify separate frontend files and may proceed in parallel after their tests.
- Never parallel-edit shared ../zunera-frontend/src/views/reports/ReportsView.vue, ../zunera-frontend/src/stores/reports/reportsStore.js, ../zunera-frontend/e2e/financial-reports.spec.js, ../zunera-backend/routes/api.php, or shared recognition services.

### Parallel example: US2

    Task: T033 test interval labels and numeric alternative in ../zunera-frontend/src/components/reports/__tests__/ReportEvolution.spec.js
    Task: T034 test ranked income/expense categories in ../zunera-frontend/src/components/reports/__tests__/ReportCategoryBreakdown.spec.js
    After tests: T035 implement ReportEvolution.vue and T036 implement ReportCategoryBreakdown.vue in separate files.

### Parallel example: US5

    Task: T049 test account movements in ../zunera-frontend/src/components/reports/__tests__/ReportAccountActivity.spec.js
    Task: T050 test filter chips/reset in ../zunera-frontend/src/components/reports/__tests__/ReportFilterBar.spec.js
    After tests: T051 implement ReportAccountActivity.vue and T052 implement ReportFilterBar.vue in separate files.

## Implementation Strategy

### MVP first

1. Finish Setup and the full backend gate T001–T025; the shared contract cannot be safely split by UI story.
2. Deliver US1 T026–T032. Verify realized summary, authorization, and correct money under its independent test.
3. Stop at the US1 checkpoint if releasing an MVP. Later story components expand analysis without changing the accounting definition.

### Incremental delivery

1. Add US2 evolution/categories, then US3 source traceability.
2. Add US4 comparison, US5 account/filter analysis, then US6 full period controls and empty states.
3. Run Phase 9 once chosen scope is complete. Reopen failing money/authorization gates before advancing.

## Notes

- [P] marks distinct files with no incomplete direct dependency; it is an execution option, not a requirement to use multiple agents.
- Every source amount remains integer BRL centavos until UI formatting. Refund recognition changes shared backend projections; no Reports-only override.
- No new package, report balance, reporting ledger, or goal-performance view is included.
- Test commands and acceptance fixtures are in specs/016-financial-reports/quickstart.md. Do not run frontend implementation before T025.
