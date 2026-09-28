# Tasks: Financial Reports Revision

**Branch**: `017-financial-reports` in specs, backend, and frontend. The feature directory remains `specs/016-financial-reports/`.
**Input**: [spec.md](spec.md), [plan.md](plan.md), [research.md](research.md), [data-model.md](data-model.md), [contract](contracts/financial-reports-api.yaml), and [quickstart.md](quickstart.md).
**Baseline**: The delivered task set is preserved in [tasks-baseline-2026-09-26.md](tasks-baseline-2026-09-26.md). It records T001–T065 and T067 complete; its pending participant study is carried into T026 below. This file contains only work for the revised specification.
**Tests**: Required for the changed source-revision contract and critical report-to-detail journey. For each “add or verify” task, add the missing assertion or cite the existing assertion and its passing result in specs/016-financial-reports/checklists/revision-audit.md; do not duplicate equivalent tests.
**Delivery order**: Complete all backend contract, authorization, validation, consistency, and story acceptance tests in Phase 2, then pass the final backend gate before any frontend work.

## Phase 1: Setup

**Purpose**: Establish the revision baseline without rebuilding the shipped feature.

- [ ] T001 Verify `017-financial-reports` is active in specs, backend, and frontend; record branch and clean application starting states in specs/016-financial-reports/checklists/revision-audit.md.
- [ ] T002 Map FR-003, FR-013, FR-025, FR-031–FR-037 and SC-001–SC-006 to existing code/tests and open gaps in specs/016-financial-reports/checklists/revision-audit.md, using specs/016-financial-reports/tasks-baseline-2026-09-26.md as delivered history.

**Checkpoint**: Only confirmed gaps proceed to implementation; no package or reporting ledger is added.

---

## Phase 2: Backend Contract, Story Checks, and Final Gate

**Purpose**: Make an overview and a later contribution read identify the same authoritative source state. This blocks all frontend revision work.

- [ ] T003 [P] Add a failing concurrent-source-change overview test that exposes mixed summary, interval, category, account, or comparison states in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportOverviewContractTest.php.
- [ ] T004 [P] Add failing contract tests for matching and changed `source_revision`, including two offsetting contributor edits with an unchanged total, a report-visible description or identity-label edit with unchanged financial effect, a prior-period edit, foreign data isolation, a changed card refund, and a concurrent source change during one detail read in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportContributionsContractTest.php.
- [ ] T005 Implement an opaque, scope-bound revision derived from relevant authoritative contribution and account-movement identities, financial effects, and report-visible explanatory fields in ../zunera-backend/app/Services/FinancialReports/ReportSourceRevisionService.php; include current and comparison periods and avoid a persisted report model.
- [ ] T006 [P] Make overview sections and their `source_revision` come from one coherent source state in ../zunera-backend/app/Services/FinancialReports/FinancialReportService.php; retain null/unavailable section semantics and centavo reconciliation.
- [ ] T007 [P] Return each bounded detail page's source rows, all-record total, and scope-bound `source_revision` from one coherent source state in ../zunera-backend/app/Services/FinancialReports/FinancialReportContributionService.php; ensure a source change between pages is detectable and verify the concurrent-detail-read case from T004.
- [ ] T008 [US1] Add or verify a focused summary-to-contribution revision and Dashboard-parity assertion, including an effective future-dated transaction in a custom range, in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportOverviewContractTest.php.
- [ ] T009 [US2] Add or verify zero-activity and partial-boundary interval, category-sum, and refund reconciliation cases under one overview revision in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportOverviewContractTest.php. Confirm current required-category behavior; if an authoritative uncategorized source is valid, also verify its null-identity distribution/comparison row and no-ID category contribution detail against specs/016-financial-reports/contracts/financial-reports-api.yaml.
- [ ] T010 [US4] Add or verify previous-period source-revision and zero/negative/unequal-duration comparison assertions in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportOverviewContractTest.php.
- [ ] T011 [US5] Add or verify account-attributed income/expense, transfer direction, settlement separation, and movement-only revision changes in ../zunera-backend/tests/Feature/FinancialReports/FinancialReportAccountActivityTest.php.
- [ ] T012 [US6] Add or verify current/previous/custom/historical boundaries, Sao Paulo business-date rollover, valid future-effective custom activity, and invalid range handling in ../zunera-backend/tests/Unit/FinancialReports/ReportPeriodResolverTest.php and ../zunera-backend/tests/Feature/FinancialReports/FinancialReportAuthorizationTest.php.
- [ ] T013 Run the final backend gate for the 1.1.0 contract in specs/016-financial-reports/contracts/financial-reports-api.yaml after T003–T012: focused Reports/card/Dashboard/Budget tests, full Laravel suite, MySQL concurrent-change cases, and Pint; record commands and results in specs/016-financial-reports/checklists/revision-verification.md before any frontend edit.

**Checkpoint**: Protected report reads return a tested source revision; overview and each detail page are coherent; all backend story acceptance and recognition tests pass before frontend work.

---

## Phase 3: User Story 1 - Understand a Period's Result (Priority: P1) 🎯 MVP

**Goal**: Keep realized income, expenses, and result accurate under the revised source-state contract.

**Independent Test**: Effective ordinary and card fixtures reconcile three summary values to the cent, exclude transfers/goals/pending sources, and carry the same revision as their contribution detail.

- [ ] T014 [US1] Verify the existing realized summary, signed result meaning, and summary contribution action consume the verified contract in ../zunera-frontend/src/components/reports/__tests__/ReportSummary.spec.js; change ../zunera-frontend/src/components/reports/ReportSummary.vue only if the acceptance check fails.

**Checkpoint**: The revised contract still supports a complete, independently testable realized summary.

---

## Phase 4: User Story 2 - Explore Evolution and Categories (Priority: P1)

**Goal**: Preserve a complete numerical trend and category composition under one source state.

**Independent Test**: Daily/weekly/monthly interval sums and income/expense category totals equal summary; zero intervals remain accessible and a refund adjusts its original expense category once.

- [ ] T015 [US2] Verify that the full interval breakdown includes zero buckets and readable numeric values in ../zunera-frontend/src/components/reports/__tests__/ReportEvolution.spec.js and ../zunera-frontend/src/components/reports/ReportDetailedBreakdown.vue; edit the component only for a confirmed gap.

**Checkpoint**: Users can explain the summary from interval and category values without relying on a chart.

---

## Phase 5: User Story 3 - Trace a Reported Amount (Priority: P1)

**Goal**: Refresh overview and contribution detail automatically when source records change, even if their net total does not.

**Independent Test**: Open a category amount, make offsetting contributor edits, then open detail; a brief notice appears and both refreshed views retain the period/filters, share one revision, and reconcile across pages.

- [ ] T016 [P] [US3] Test the new overview and detail `source_revision` response shapes and cancellation behavior in ../zunera-frontend/src/services/__tests__/reportsService.spec.js and ../zunera-frontend/src/services/__tests__/reportsContributionService.spec.js.
- [ ] T017 [P] [US3] Write failing store tests for matching revisions, changed totals, net-zero contributor substitutions, report-visible label edits, scope changes during refresh, later-page revision changes, bounded retries, and recoverable failure in ../zunera-frontend/src/stores/reports/__tests__/reportsStore.spec.js.
- [ ] T018 [US3] Implement revision comparison, automatic same-scope overview/detail refresh, bounded retry, and stale-page rejection in ../zunera-frontend/src/stores/reports/reportsStore.js after T016–T017 and the Phase 2 backend gate.
- [ ] T019 [US3] Show the brief change notice while preserving the contribution target and active report context in ../zunera-frontend/src/views/reports/ReportsView.vue and ../zunera-frontend/src/i18n/messages.js; keep the drawer's existing signed source rows and paging in ../zunera-frontend/src/components/reports/ReportContributionDrawer.vue.
- [ ] T020 [US3] Verify notice, preserved scope, reconciled detail, and bounded-retry error presentation in ../zunera-frontend/src/views/reports/__tests__/ReportsView.spec.js.
- [ ] T021 [US3] Add a critical browser journey for changed-value and unchanged-total contributor edits, filter preservation, and source-revision agreement in ../zunera-frontend/e2e/financial-reports.spec.js.

**Checkpoint**: Contributions remain explainable after a source change; no stale overview/detail pair is presented as current.

---

## Phase 6: User Story 4 - Compare Equivalent Periods (Priority: P2)

**Goal**: Keep prior-period calculations and labels correct when prior sources change.

**Independent Test**: A prior-period edit changes the shared report revision and refreshes the comparison; exact ranges, absolute differences, and unavailable percentages remain correct.

- [ ] T022 [US4] Verify current/prior labels and percentage-unavailable reasons after a revision refresh in ../zunera-frontend/src/components/reports/__tests__/ReportComparison.spec.js; edit ../zunera-frontend/src/components/reports/ReportComparison.vue only for a confirmed gap.

**Checkpoint**: Comparison is independently testable with a changed prior period and an unchanged selected scope.

---

## Phase 7: User Story 5 - Analyze Accounts and Filter Scope (Priority: P2)

**Goal**: Keep direct flows, transfers, and card settlements separate under account/category/type filters and source changes.

**Independent Test**: An account movement edit changes the report revision without changing consolidated income/expense; filtered overview and account detail still agree.

- [ ] T023 [US5] Verify account/category/type filters remain visible and unchanged through automatic refresh, while transfer and settlement detail stay suppressed under type/category filters, in ../zunera-frontend/src/components/reports/__tests__/ReportAccountActivity.spec.js and ../zunera-frontend/src/components/reports/__tests__/ReportFilterBar.spec.js.

**Checkpoint**: Account analysis explains cash movement without changing consolidated result.

---

## Phase 8: User Story 6 - Navigate Periods and Empty States (Priority: P2)

**Goal**: Keep period boundaries, empty states, and restored context correct during revision refresh.

**Independent Test**: Previous month equals last completed month; a custom future range honors an already effective transaction; current presets stop today; empty and unavailable sections remain distinct after a refresh.

- [ ] T024 [US6] Verify previous-month/last-completed-month wording, custom interval restoration, and empty versus unavailable state after automatic refresh in ../zunera-frontend/src/components/reports/__tests__/ReportPeriodSelector.spec.js and ../zunera-frontend/src/views/reports/__tests__/ReportsView.spec.js.

**Checkpoint**: Every section and contribution view retains one clearly labelled period/filter context.

---

## Phase 9: Polish and Cross-Cutting Outcome Evidence

**Purpose**: Complete remaining measurable outcomes and final verification.

- [ ] T025 With 10,000 records, 100 categories, and 50 accounts, measure at least 20 complete live attempts for each representative displayed-total class: summary income/expense/result, income/expense categories including a small contributor, account direct flow and separate movements, and current/prior comparison. Include first-page and deeper-page contribution paths. Record per-class and overall shares reaching the requested records within five seconds, plus revision-calculation cost, in specs/016-financial-reports/checklists/performance.md. If any sampled class misses 95% or a deeper page is unreachable, identify the bottleneck, make a focused fix in the affected application, rerun relevant tests and the same live measurement, and keep this task open until SC-005 is met.
- [ ] T026 Run the pending uncoached study with at least 10 representative participants using fictional data, and record anonymized SC-002/SC-003 pass rates in specs/016-financial-reports/checklists/usability.md. If either outcome misses 90%, identify the observed usability obstacle, make a focused improvement in the affected experience, rerun relevant tests and an equivalent uncoached study, and keep this task open until both thresholds are met.
- [ ] T027 Reconcile delivered source-revision behavior, response shape, and manual acceptance steps in specs/016-financial-reports/plan.md, specs/016-financial-reports/data-model.md, specs/016-financial-reports/contracts/financial-reports-api.yaml, and specs/016-financial-reports/quickstart.md.
- [ ] T028 After T025–T026 meet their thresholds, run final backend/frontend test, style, build, and isolated Playwright commands from specs/016-financial-reports/quickstart.md; record results and spec/contract parity in specs/016-financial-reports/checklists/revision-verification.md.

**Checkpoint**: Financial values, source records, and period/filter context agree; recorded evidence meets SC-002, SC-003, and SC-005. If an outcome remains unmet or its study cannot run, keep the corresponding task and final gate open and report the blocker without claiming completion.

---

## Dependencies and Execution Order

### Phase dependencies

1. Phase 1 establishes the revision baseline. T003 and T004 may run in parallel; T005 follows both. T006 and T007 may run in parallel after T005. Backend story checks T008–T012 follow the relevant source-state work; T008–T010 share one test file and run sequentially. T013 closes the backend gate only after every backend task T003–T012 passes.
2. No frontend task T014–T024 starts before T013. Each story consumes its completed backend acceptance check from Phase 2.
3. US1, US2, US4, US5, and US6 are independently testable after T013. US3 also needs T016 and T017 before T018; T019 and T020 follow T018, and T021 follows both.
4. Phase 9 follows the required story checkpoints. T025 and T026 can collect evidence independently; coordinate any resulting application edits. T027 follows implementation and evidence review, and T028 follows successful T025–T027.

### User-story dependency graph

```text
Setup → Backend source revision + all story checks → Final backend gate
                                                ├→ US1: realized summary
                                                ├→ US2: evolution and categories
                                                ├→ US3: contribution refresh
                                                ├→ US4: prior comparison
                                                ├→ US5: account and filter scope
                                                └→ US6: period and empty states
All selected stories → outcome evidence → final verification
```

### Parallel opportunities and independent checks

| Story | Independent check | Safe parallel work after dependencies |
|---|---|---|
| US1 | Summary, Dashboard parity, and contribution revision reconcile. | Backend T008 and T011 use different files; frontend T014 starts after T013. |
| US2 | All intervals and categories reconcile, including zero buckets. | Frontend T015 can run alongside US4 T022 after T013. |
| US3 | Changed contributors trigger automatic coherent refresh. | T016 and T017 use different frontend test files; T018 waits for both. |
| US4 | Previous-period edits refresh comparison without false percentages. | Frontend T022 can run alongside US2 T015 after T013. |
| US5 | Movements stay separate under filters and revision changes. | Backend T011 can run alongside T008–T010 in different files. |
| US6 | Date boundaries and empty/unavailable states remain correct. | Backend T012 can run alongside T011 in different files. |

Do not parallel-edit ../zunera-backend/app/Services/FinancialReports/FinancialReportService.php, ../zunera-frontend/src/stores/reports/reportsStore.js, ../zunera-frontend/src/views/reports/ReportsView.vue, or ../zunera-frontend/e2e/financial-reports.spec.js. Backend T008–T010 share the overview contract test file and are sequential.

## Implementation Strategy

### MVP first

1. Finish T001–T013, including all backend story checks and the final protected 1.1.0 contract gate.
2. Finish US1 T014 and verify its independent summary test. This retains the existing user-visible MVP while adding trustworthy source-state metadata.
3. Complete US3 T016–T021 before treating source-change explainability as delivered; it is the main new user behavior in this revision.

### Incremental delivery

1. After the backend gate, close US2, US4, US5, and US6 frontend acceptance checks against the existing implementation; fix only demonstrated gaps.
2. Meet SC-005 live latency and SC-002/SC-003 participant thresholds, then T027–T028 final parity checks.
3. Keep the original feature task history in tasks-baseline-2026-09-26.md; mark only this revision's tasks complete here.

## Notes

- `[P]` means different files with no unfinished direct dependency. It does not require multiple agents.
- No new package, report financial record, balance model, or goal-performance view is included.
- Backend read revisions are metadata from authoritative sources. All money remains exact centavos until display formatting.
