# Financial Reports revision audit

**Branch check (T001, 2026-09-27):** `017-financial-reports` is active in specs, backend, and frontend. All three worktrees were clean before this revision began. The established feature directory remains `016-financial-reports`. Existing ignore files cover generated dependencies, environment files, logs, builds, and test output; no setup file needs changing.

## Delivered baseline and open revision gaps (T002)

The completed baseline is recorded in [tasks-baseline-2026-09-26.md](../tasks-baseline-2026-09-26.md). At the start of this revision, report reads used `FinancialReportService`, `FinancialReportContributionService`, `RecognizedContributionRepository`, and `AccountMovementRepository`. The overview read sections independently; contribution detail read its all-record total and page independently. Neither response then included `source_revision`.

| Requirement | Existing implementation and evidence | Revision gap / task |
|---|---|---|
| FR-003, FR-035 | `ReportPeriodResolverTest` covers historical and custom calendar comparisons; `FinancialReportAuthorizationTest` covers invalid presets, dates, and filters. | Verify business-date rollover and valid future-effective custom activity (T012). |
| FR-013, FR-031 | `FinancialReportService::evolution` emits complete daily, weekly, or monthly intervals; `ReportEvolution` and `ReportDetailedBreakdown` provide numeric values. | Add or cite focused zero-activity and partial-boundary assertions (T009, T015). |
| FR-025 | `FinancialReportContributionsContractTest` covers signed refund trace, cursor pages, all-record total, and movement separation. | Make page rows and total coherent; detect changes across requests and pages (T004, T007, T016–T021). |
| FR-032 | Existing source rules require categories for eligible ordinary and card activity; `data-model.md` and API contract describe null identity if a valid uncategorized source appears. | Confirm source rule; add null-identity assertion if applicable (T009). |
| FR-033 | Current overview and detail have no shared source revision or read snapshot. | Concurrent-read tests, source revision, coherent responses, automatic frontend refresh (T003–T007, T017–T021). |
| FR-034 | Integer centavos and server-derived money are used in existing report services; browser components format for display. | Verify amounts and formatting through revision refresh (T008–T024). |
| FR-036 | URL-backed report scope and current filter/detail tests preserve context. | Verify scope survives automatic revision refresh (T017, T019–T024). |
| FR-037, SC-001 | `FinancialReportOverviewContractTest`, `ReportRecognitionParityTest`, and card/Dashboard/Budget tests cover recognized totals and paid refunds. | Reverify summary/detail parity under one source state (T008–T013). |
| SC-002, SC-003 | Automated interface checks exist, but uncoached participant evidence is absent. | Run participant study; record results (T026). |
| SC-004 | Period resolver, authorization, and browser navigation tests cover boundaries and labels. | Recheck scope and comparison after refresh (T012, T022, T024). |
| SC-005 | Separate MySQL and mocked-browser timings are recorded in `performance.md`; integrated live target is unproven. | Run integrated per-class latency study (T025). |
| SC-006 | `ReportPeriodResolverTest` covers zero, negative, sign-crossing, and unequal-day percentage suppression. | Recheck comparison after revision refresh (T010, T022). |

For each “add or verify” task, this audit will receive the specific test assertion and passing command when the task is closed. Baseline coverage alone does not close a revision task.

## Completed backend source-state work

- T003–T007: `FinancialReportConcurrentSnapshotTest` proves MySQL overview and detail keep their original source state when a second connection commits an amount change between internal reads. Command: `php artisan test --filter=FinancialReportConcurrentSnapshotTest` with `.env.testing` (2 tests, 18 assertions passed).
- `FinancialReportContributionsContractTest` proves overview/detail revision agreement, offsetting edits with unchanged total, prior-period category-label changes, foreign isolation, refund changes, scope binding, and stale later-page detection. `FinancialReportOverviewContractTest` proves unchanged-amount description edits change revision. `FinancialReportAccountActivityTest` proves movement-only edits change revision. Focused revision/refund/movement tests passed in the Docker container.
- T008: `FinancialReportOverviewContractTest::effective_future_dated_custom_activity_reconciles_with_detail_and_dashboard` proves custom future-effective contribution, Dashboard parity, and matching revision (passed).
- T009: `FinancialReportOverviewContractTest::weekly_partial_boundaries_and_zero_intervals_reconcile_under_one_revision`, existing paid-refund case, and existing empty-state case cover zero intervals, partial boundaries, category sums, and refunds (passed). Existing Transaction and card source validation require categories; no valid uncategorized source was found.
- T010: `FinancialReportContributionsContractTest::revision_tracks_prior_period_labels_and_excludes_foreign_sources` proves a prior-period label changes revision; existing `ReportPeriodResolverTest` covers zero, negative, sign-crossing, and unequal durations (passed).
- T011: `FinancialReportAccountActivityTest` covers attributed direct flow, transfer directions, settlement separation, filter suppression, and movement-only revision changes (passed).
- T012: `ReportPeriodResolverTest` covers current, completed, custom, historical, leap/short month, and UTC-to-Sao-Paulo midnight rollover. `FinancialReportOverviewContractTest` covers valid future-effective custom activity. `FinancialReportAuthorizationTest` covers invalid dates/ranges and foreign IDs (passed).

## Completed frontend story checks

- T014–T015: existing `ReportSummary.spec.js` and `ReportEvolution.spec.js` passed (6 tests). Summary meaning/detail action and the complete numerical interval breakdown needed no component changes.
- T016–T020: service tests passed (6 tests), store tests passed (9 tests), and `ReportsView.spec.js` passed (5 tests). These cover matching/mismatched revisions, unchanged totals with changed contributors, changed labels, scope changes during refresh, later-page rejection, bounded recovery, notice, and refreshed section state.
- T021: the full isolated `e2e/financial-reports.spec.js` suite passed (51 tests across Chromium, Firefox, and WebKit). New changed-amount and unchanged-total browser cases preserve the custom category scope and show matching refreshed revisions.
- T022–T024: `ReportComparison`, `ReportAccountActivity`, `ReportFilterBar`, and `ReportPeriodSelector` component tests passed (7 tests); `ReportsView.spec.js` checks the refreshed comparison, movement suppression, custom period context, and unavailable section against a reliable sibling.
- T025: Live browser/API/MySQL measurement passed 260/260 complete attempts across 13 displayed-total and paging paths on a disposable 10,000-record, 100-category, 50-account fixture. Revision cost and per-class results are in [performance.md](performance.md). T026 remains open for uncoached participants; T028 waits for it.
