# Implementation Plan: Financial Reports

**Branch**: `017-financial-reports` | **Date**: 2026-09-27 | **Spec**: [spec.md](spec.md)
**Input**: Revised functional specification and the existing Reports implementation in `specs/016-financial-reports/`

**Branch Coordination**: `017-financial-reports` is active in specs, backend, and frontend as of this revision. Recheck before editing either application. The established feature directory remains `016-financial-reports`; directory and branch names are independent.

## Summary

Financial Reports has a read-only overview, period and filter controls, evolution with detailed numerical breakdown, separate income and expense categories, account activity, comparison, and paged contribution detail. This revision added a scope-bound source revision to both protected reads. Each response uses one database read snapshot; the browser refreshes the overview and first detail page under the same scope when revisions differ, with three bounded attempts and a recoverable retry state. The shared card-recognition repair and backend-first gate remain complete. SC-005 passed the live 10,000-record measurement; the uncoached participant outcomes remain open in [tasks.md](tasks.md).

## Technical Context

**Language/Version**: PHP 8.3 with Laravel `^13.17`; JavaScript with Vue `^3.5.40` and `<script setup>` (current project convention)  
**Primary Dependencies**: Existing Sanctum, Eloquent/query builder, Axios, Pinia `^4.0.2`, Vue Router `^5.2.0`, Element Plus `^2.14.5`, Tailwind, ApexCharts `^7.4.0`, vue-i18n; no new package  
**Storage**: Existing MySQL development database and SQLite tests; source financial records only; exact integer BRL centavos; no report persistence planned  
**Testing**: Laravel feature/unit/contract tests, MySQL card-refund integration regression, PHPUnit suite and Pint; Vitest service/store/component tests, ESLint, Playwright browser journeys  
**Target Platform**: Authenticated, responsive Zunera web app; America/Sao_Paulo financial business date; PT-BR and English formatting  
**Project Type**: Full-stack feature across separate backend/frontend repositories  
**Performance Goals**: At 10,000 owned financial records, 100 categories, and 50 accounts, summary and access to any contribution complete within 5 seconds in at least 95% of normal test attempts  
**Constraints**: Owner-scoped reads, current financial-recognition rules, no expected panel, no duplicate card expense, no new balance model, exact centavos, no new secrets/uploads/packages  
**Scale/Scope**: Current/previous month and year shortcuts, completed historical-month navigation, inclusive custom dates, three optional financial filters, one coherent overview, and paged detail. “Last completed month” is the existing previous-month choice, not another period mode.

## Delivery Scope and Order

1. **Backend** (`../zunera-backend`): Audit the existing protected read contract, period resolution, recognition, interval completeness, contribution reconciliation, account attribution, and authorization against the revised spec. If a gap is confirmed, update the authoritative projection or report read contract with focused tests before frontend changes. The paid-source refund repair is already implemented.
2. **Frontend** (`../zunera-frontend`): Audit the existing URL-backed scope, period naming, numerical breakdown, contribution context, and accessible states against verified backend behavior. Implement only confirmed gaps and add focused service, component, or browser coverage.

The original backend gate passed before the existing frontend implementation. Any new backend-dependent frontend work must repeat the relevant gate first.

## Constitution Check

*Revision pre-research gate: PASS on 2026-09-27. Revision post-design gate: PASS on 2026-09-27.*

- [x] API boundary uses existing `auth:sanctum` and `session.lifetime`, owner-scoped resolution, Form Requests for all query inputs, API Resources for output, and safe validation/404 errors. No upload surface.
- [x] Backend card-recognition correction and Report contract/tests precede frontend work; frontend consumes the documented response and error shapes.
- [x] Identical `017-financial-reports` branch was verified in specs, backend, and frontend before revision planning. No application file is edited by this plan.
- [x] Financial semantics live in `App\Services`; a scoped read repository is justified by shared cross-source aggregation and paged detail. Multi-field report scope uses a DTO/value object. No global service state.
- [x] Vue plan uses Composition API with `<script setup>`, service-only API access, Pinia for shared report scope/read state, Element Plus controls, Tailwind/semantic tokens, and existing chart wrappers.
- [x] Existing backend money, ownership, date, filter, contract, refund, and comparison tests and frontend service/store/component and isolated Playwright journeys are preserved; any delta receives proportionate coverage.
- [x] No new secrets, uploads, packages, reporting ledger, or unrelated refactor. The earlier refund fix remains the shared recognition authority.
- [x] No constitution exception is needed.

Post-design recheck: the read-only contract keeps one owner-scoped report context; detail reuses signed source effects; frontend renders server-derived money; branch names match; backend verification still precedes any dependent frontend delta. Gates remain passed.

## Research and Design Decisions

- [Research](research.md) records source-rule reuse, paid-refund recognition repair, read projection, period/calendar rules, account separation, interface shape, Vue patterns, and test gate. Current framework guidance was checked in [Laravel 13 query/pagination docs](https://laravel.com/docs/13.x/queries) and [Vue watcher docs](https://vuejs.org/guide/essentials/watchers).
- [Data model](data-model.md) defines one validated report scope, comparison dates, virtual signed contributions, account movement, derived aggregates, and reconciliation invariants. No report table or report-persistence migration is planned; source-table indexes remain conditional on measured need.
- [API contract](contracts/financial-reports-api.yaml) defines a protected overview and paged contributions read with explicit period/filter metadata, centavos, empty states, signed detail, validation, and ownership errors.
- [Quickstart](quickstart.md) gives backend-first verification and an acceptance walkthrough.
- `AGENTS.md` already points to `specs/016-financial-reports/plan.md` inside its Spec Kit markers. This repository has no agent-context update script, so the correct reference is retained without rewriting unrelated guidance.
- The existing shared recognized-card projection is authoritative for Reports, Dashboard, Budgets, and card history. `FinancialHistoryService::totals()` remains unsuitable for Report summary because it omits card installments.
- The revision audit confirms that previous month already means last completed month; daily/weekly/monthly evolution emits zero buckets; the frontend exposes complete numerical interval detail; and the protected detail read supports summary/category/account metrics. No second period mode, chart system, or reporting ledger is warranted.
- [Research](research.md) records the coherent-snapshot and cross-request freshness risk addressed by this revision. A read-only source revision now covers offsetting contribution and visible-label changes that an unchanged detail total cannot detect. MySQL concurrent-write tests verify each response's read snapshot; the browser compares revisions across responses.

## Original Implementation Sequence (delivered baseline)

Steps 1–8 below describe the delivered feature and are retained as implementation history. [Baseline tasks](tasks-baseline-2026-09-26.md) records T001–T065 and T067 complete; its participant study T066 is carried into revision task T026. The integrated performance outcome passed in revision task T025. [Revision tasks](tasks.md) tracks the remaining work.

1. **Reconcile credit-event recognition**: Add a shared recognized-card allocation that assigns every accepted source-purchase credit-event centavo to original installments in sequence regardless of source statement payment state. Keep statement obligation and card-credit applications unchanged. Replace spending reads that currently equate `credit_adjustment_centavos` with full recognized refund effect. Prove paid, partial, unpaid, multi-installment, and cancellation cases against Spec 010 in card, Dashboard, Budget, and Financial History tests.
2. **Backend scope and security**: Define `ReportScope` DTO and period resolver for the five quick choices plus selected completed historical-month navigation, prior period, day counts, timezone, filter ownership, type/category compatibility, and maximum valid date bounds consistent with existing transactions. A navigated historical month compares with its full preceding month; a custom range always compares with the preceding equal-day range. Add authenticated report routes, thin controller(s), query Form Requests, Resources, and domain errors. Reject foreign IDs without disclosure.
3. **Backend contribution read model**: Build one owner-scoped repository/query layer for effective ordinary transactions and realized net card installments plus individually traceable negative credit-event contributions. Apply account/category/type filters consistently. Preserve archived identities and original recognized dates. Aggregate summary, evolution, category totals/shares, and current/prior comparison from that layer; compare Dashboard under equivalent scope. Report independently unavailable sections as null with explicit section state, never as valid zero data.
4. **Backend account movement**: Add per-account attributed income/direct expense and net flow, plus separately labelled effective incoming/outgoing transfers and card statement settlement. Account filters exclude unattributed card spending. Type/category filters suppress unrelated transfer/settlement figures rather than reclassifying them. Make mismatch between account direct-expense sum and workspace total explicit in response/presentation.
5. **Backend drill-down and performance**: Provide stable, bounded cursor pages and all-record totals for summary, category, account, transfer, and settlement metrics. Include source IDs, original card purchase/statement/credit event, recognized date, and signed contribution. Ensure each detail metric sums to its overview amount, including pages not initially loaded. Review query plans/indexes using 10,000-record fixture; add source-table indexes only if necessary.
6. **Complete backend gate**: Run feature/unit/contract tests for all spec acceptance cases, authentication, foreign IDs, invalid range/filter, period boundaries/leap cases, unequal-day percent suppression, zero/negative comparisons, pending/recurring/goal exclusions, archived history, refunds/corrections, and paged reconciliation. Run full suite and Pint inside development container; verify MySQL-specific recognition and report behavior. Frontend starts only when this gate passes.
7. **Frontend Reports**: Add protected route/nav and i18n; establish the canonical URL-backed scope and request guards with the US1 route/store foundation before detail and filter integrations. Later add controls for every period mode and filter without replacing that scope model. Compose summary, evolution, separate category sections, account table, comparison, filters, period selector, and report-specific contribution drawer. Reuse existing formatters and chart/theme wrapper with visible numbers/text alternatives.
8. **Frontend verification and outcomes**: Test URL restoration, rapid filter/period changes without mixed-context sections, no-data/no-income/no-expense/no-prior-data states, signed card adjustment detail, account movement labels, keyboard/screen-reader and color-independent meaning, 320 px/200% zoom, light/dark/system, PT-BR/English, and Playwright report-to-detail/filter/comparison journeys. Measure 10,000-record response/navigation target and uncoached SC-002/SC-003 tasks; record results for review.

## Revision Implementation Sequence

1. **Backend specification audit**: Re-run the protected overview and contribution contract fixtures against FR-003, FR-013, FR-025, FR-031–FR-037. Confirm previous-month alias, inclusive custom and future-effective behavior, zero intervals, signed refund contributions, category reconciliation, and owner isolation. Add a test only for a behavior or regression risk not already covered.
2. **Coherent read and revision**: Exercise a financial record change while an overview is assembled and during one contribution-detail read. Make every overview and each detail response use a coherent financial source state without persisting a report model; keep section-level unavailable states distinct from valid zero. Each detail page's rows, all-record total, and source revision must describe that one state; later pages may observe a newer state but must expose a changed revision before their rows join the displayed set. Add an opaque source revision to overview and detail that changes when relevant source identity, recognition state, date, classification, amount, or report-visible identifying/explanatory information changes, including net-zero contributor substitutions and same-amount label edits. Include relevant current/prior-period sources and separate account movements. Verify in MySQL with focused tests. Keep account movements separate from consolidated income and expenses.
3. **Backend gate for changed contract**: Update [data-model.md](data-model.md) and [the contract](contracts/financial-reports-api.yaml) for source revision. Test same-revision reads, changed-value and unchanged-total source changes, same-amount visible-information edits, ownership, period/filter scope, refund and transfer effects, concurrent detail reads, and bounded detail paging. Complete all backend story acceptance checks, then run focused report/card/Dashboard/Budget coverage, full backend suite, and Pint before any frontend edit.
4. **Frontend consistency**: Preserve current scope through all sections and detail, including the zero-interval breakdown. Compare overview and detail source revisions; on mismatch, automatically refresh both under the same period and filters, show a brief notice, and display them only when revisions agree. Bound automatic retries and offer a recoverable retry state during continuing changes. Cover net-zero contributor changes and rapid filter changes in service/store/component/browser checks.
5. **Outcome evidence**: Measure complete live overview-to-contribution journeys at the 10,000-record/100-category/50-account scale with at least 20 attempts per representative displayed-total class. Include summary, category (including a small contributor), account direct flow/separate movements, current/prior comparison, and first/deeper detail pages; record per-class and overall results in [performance.md](checklists/performance.md). Run the pending uncoached 10-participant study for SC-002/SC-003 and record anonymized outcomes in [usability.md](checklists/usability.md). These measurements are not inferred from isolated backend and mocked-browser timings. If any sampled class misses SC-005 or SC-002/SC-003 misses its threshold, identify the observed cause, make a focused correction, and repeat the affected test or study; keep the outcome and final delivery gate open until the threshold is met.
6. **Final verification**: Recheck critical report-to-detail, filter, comparison, period-boundary, and zero-activity journeys, along with responsive/theme/accessibility behavior. Confirm the spec, plan, contract, quickstart, and task status describe the same delivered behavior. Run relevant commands in [quickstart.md](quickstart.md).

## Project Structure

### Documentation (this feature)

```text
specs/016-financial-reports/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/financial-reports-api.yaml
├── tasks.md
├── tasks-baseline-2026-09-26.md
└── checklists/{requirements,revision-audit,revision-verification,performance,usability}.md
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/Data/FinancialReports/
├── app/Http/Controllers/Api/V1/FinancialReportController.php
├── app/Http/Requests/FinancialReports/
├── app/Http/Resources/FinancialReports/
├── app/Repositories/FinancialReports/
├── app/Services/FinancialReports/
├── app/Services/FinancialDashboard/RecurringCardExpenseProjection.php
├── app/Services/CreditCards/
├── app/Services/Budgets/
├── routes/api.php
└── tests/{Feature,Unit}/FinancialReports/

../zunera-frontend/
├── src/router/index.js
├── src/layouts/AppShell.vue
├── src/views/reports/
├── src/components/reports/
├── src/composables/reports/
├── src/services/reportsService.js
├── src/stores/reports/
├── src/i18n/
└── e2e/financial-reports.spec.js
```

**Structure Decision**: Retain the existing folders and JavaScript conventions. Shared card recognition remains in the card/dashboard/budget domain, with no Reports-only override, new library, or reporting table.

## Frontend Component Map

### Existing Reports composition and revision boundary

Keep the existing API and URL scope. `ReportsView` composes period and filter controls, summary, evolution, categories, account activity, and comparison. `ReportEvolution` delegates the full numerical interval view to `ReportDetailedBreakdown`. Component-local disclosure remains local; shared fetched scope stays in Pinia. Preserve the existing contribution event contract. Verify FR-031–FR-036 and EX-001–EX-003 through focused component and browser checks.

| Component | Single responsibility | Inputs / outputs |
|---|---|---|
| `ReportsView` | Compose sections and coordinate one resolved scope | URL scope/store state → child props; handles child events |
| `ReportPeriodSelector` | Choose month/year shortcuts, completed historical month, or custom dates | Scope in; period-change event out |
| `ReportFilterBar` | Choose account/category/type, show chips and reset | Filter options/active filters in; filter/reset events out |
| `ReportSummary` | Show realized income, expenses, and signed result | Summary/scope in; metric-detail event out |
| `ReportEvolution` | Present labelled interval trends with numeric alternative | Interval/granularity in; no domain calculation |
| `ReportDetailedBreakdown` | Expose every interval's income, expenses, and result, including zero buckets | Intervals/locale in; no domain calculation |
| `ReportCategoryBreakdown` | Present one classification's ranked categories and shares | Income or expense rows in; category-detail event out |
| `ReportAccountActivity` | Show account income/direct expense/net plus separate movements | Account rows in; metric-detail event out |
| `ReportComparison` | Show exact periods, absolute changes, and available percentages | Comparison/scope in; previous/current detail event out |
| `ReportContributionDrawer` | Show paged signed source records and reconciliation | Metric/scope/page in; request-next/close events out |

Pinia owns only state shared by route sections and detail. Pure formatting stays in existing utilities. A scope change invalidates stale overview/detail responses; computed values derive labels and visibility without client-side monetary recalculation. Compare overview and detail source revisions, refresh both with the same scope on mismatch, and explain the change. The report drawer handles card installments/credit events directly because the Transactions detail click path skips `credit_card_expense` rows.

## Verification Gates and Risks

- **Money integrity**: The paid-source refund correction is implemented in the shared recognized-card projection. Preserve its cross-feature tests; never create a Report-only override.
- **Scope and source consistency**: One validated scope covers aggregates and drill-down, and foreign IDs reveal no ownership data. Audit whether one overview can mix source states under concurrent writes; correct any confirmed issue in the backend before frontend handling of changed detail totals.
- **Volume**: Overview is aggregated and detail paged. The live combined SC-005 measurement passed 260/260 browser/API/MySQL attempts across 13 paths; see [performance.md](checklists/performance.md). No cache was needed.
- **Accessibility**: Charts supplement, never replace, labelled values and reachable records. Follow four design documents and existing Element Plus theme behavior.
- **Constitution**: No exceptions; the read repository is justified by complex cross-source aggregation and shared reconciliation.
