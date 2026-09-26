# Implementation Plan: Financial Reports

**Branch**: `016-financial-reports` | **Date**: 2026-09-26 | **Spec**: [spec.md](spec.md)  
**Input**: Clarified feature specification from `specs/016-financial-reports/spec.md`

**Branch Coordination**: `016-financial-reports` is active in `zunera-specs`, `../zunera-backend`, and `../zunera-frontend`. Recheck before editing either application.

## Summary

Add a dedicated, read-only Reports area with realized summary, evolution, separate income/expense categories, account activity, previous-period comparison, consistent filters, and paged source-record drill-down. A single backend report scope drives all sections. Reuse the existing financial definitions and repair a discovered paid-card-refund recognition gap in shared card projections before Report calculations so Reports, Dashboard, Budgets, and card history reconcile.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel `^13.17`; JavaScript with Vue `^3.5.40` and `<script setup>` (current project convention)  
**Primary Dependencies**: Existing Sanctum, Eloquent/query builder, Axios, Pinia `^4.0.2`, Vue Router `^5.2.0`, Element Plus `^2.14.5`, Tailwind, ApexCharts `^7.4.0`, vue-i18n; no new package  
**Storage**: Existing MySQL development database and SQLite tests; source financial records only; exact integer BRL centavos; no report persistence planned  
**Testing**: Laravel feature/unit/contract tests, MySQL card-refund integration regression, PHPUnit suite and Pint; Vitest service/store/component tests, ESLint, Playwright browser journeys  
**Target Platform**: Authenticated, responsive Zunera web app; America/Sao_Paulo financial business date; PT-BR and English formatting  
**Project Type**: Full-stack feature across separate backend/frontend repositories  
**Performance Goals**: At 10,000 owned financial records, 100 categories, and 50 accounts, summary and access to any contribution complete within 5 seconds in at least 95% of normal test attempts  
**Constraints**: Owner-scoped reads, current financial-recognition rules, no expected panel, no duplicate card expense, no new balance model, exact centavos, no new secrets/uploads/packages  
**Scale/Scope**: Five quick period choices plus completed historical-month navigation, one coherent overview, comparison, four filter dimensions including period, paged detail, two authenticated read interfaces

## Delivery Scope and Order

1. **Backend** (`../zunera-backend`): First repair shared card spending recognition for paid-source refunds in line with Spec 010. Then define protected report contract, scope validation, owner-scoped recognized contributions, aggregates, comparison, account movements, paged detail, API Resources, and feature/unit/contract tests. Verify Dashboard/Budget/history parity before frontend starts.
2. **Frontend** (`../zunera-frontend`): Consume the completed backend contract for Reports route/navigation, URL-backed period and filters, summary/trend/category/account/comparison sections, paged detail, localized formatting, accessible states, and service/store/component/browser tests.

Backend contract, authorization, validation, services, and tests must pass before frontend implementation begins.

## Constitution Check

*Pre-research gate: PASS on 2026-09-26. Post-design gate: PASS on 2026-09-26.*

- [x] API boundary uses existing `auth:sanctum` and `session.lifetime`, owner-scoped resolution, Form Requests for all query inputs, API Resources for output, and safe validation/404 errors. No upload surface.
- [x] Backend card-recognition correction and Report contract/tests precede frontend work; frontend consumes the documented response and error shapes.
- [x] Identical `016-financial-reports` branch was verified in specs, backend, and frontend before planning.
- [x] Financial semantics live in `App\Services`; a scoped read repository is justified by shared cross-source aggregation and paged detail. Multi-field report scope uses a DTO/value object. No global service state.
- [x] Vue plan uses Composition API with `<script setup>`, service-only API access, Pinia for shared report scope/read state, Element Plus controls, Tailwind/semantic tokens, and existing chart wrappers.
- [x] Backend money, ownership, date, filter, contract, refund, and comparison cases plus frontend service/store/component and isolated Playwright journeys are planned.
- [x] No new secrets, uploads, packages, reporting ledger, or unrelated refactor. Refund fix is required to satisfy existing Spec 010 and cross-view reconciliation.
- [x] No constitution exception is needed.

Post-design recheck: read-only contract keeps a single owner-scoped report context; detail reuses the same signed contribution semantics; frontend components only render server-derived money; backend verification gate still precedes frontend. Gates remain passed.

## Research and Design Decisions

- [Research](research.md) records source-rule reuse, paid-refund recognition repair, read projection, period/calendar rules, account separation, interface shape, Vue patterns, and test gate. Current framework guidance was checked in [Laravel 13 query/pagination docs](https://laravel.com/docs/13.x/queries) and [Vue watcher docs](https://vuejs.org/guide/essentials/watchers).
- [Data model](data-model.md) defines one validated report scope, comparison dates, virtual signed contributions, account movement, derived aggregates, and reconciliation invariants. No report table or report-persistence migration is planned; source-table indexes remain conditional on measured need.
- [API contract](contracts/financial-reports-api.yaml) defines a protected overview and paged contributions read with explicit period/filter metadata, centavos, empty states, signed detail, validation, and ownership errors.
- [Quickstart](quickstart.md) gives backend-first verification and an acceptance walkthrough.
- Existing Dashboard recognition helpers must be made shared or delegated to the same domain projection rather than copied. `FinancialHistoryService::totals()` cannot back Report summary because it omits card installments.

## Implementation Sequence

1. **Reconcile credit-event recognition**: Add a shared recognized-card allocation that assigns every accepted source-purchase credit-event centavo to original installments in sequence regardless of source statement payment state. Keep statement obligation and card-credit applications unchanged. Replace spending reads that currently equate `credit_adjustment_centavos` with full recognized refund effect. Prove paid, partial, unpaid, multi-installment, and cancellation cases against Spec 010 in card, Dashboard, Budget, and Financial History tests.
2. **Backend scope and security**: Define `ReportScope` DTO and period resolver for the five quick choices plus selected completed historical-month navigation, prior period, day counts, timezone, filter ownership, type/category compatibility, and maximum valid date bounds consistent with existing transactions. A navigated historical month compares with its full preceding month; a custom range always compares with the preceding equal-day range. Add authenticated report routes, thin controller(s), query Form Requests, Resources, and domain errors. Reject foreign IDs without disclosure.
3. **Backend contribution read model**: Build one owner-scoped repository/query layer for effective ordinary transactions and realized net card installments plus individually traceable negative credit-event contributions. Apply account/category/type filters consistently. Preserve archived identities and original recognized dates. Aggregate summary, evolution, category totals/shares, and current/prior comparison from that layer; compare Dashboard under equivalent scope. Report independently unavailable sections as null with explicit section state, never as valid zero data.
4. **Backend account movement**: Add per-account attributed income/direct expense and net flow, plus separately labelled effective incoming/outgoing transfers and card statement settlement. Account filters exclude unattributed card spending. Type/category filters suppress unrelated transfer/settlement figures rather than reclassifying them. Make mismatch between account direct-expense sum and workspace total explicit in response/presentation.
5. **Backend drill-down and performance**: Provide stable, bounded cursor pages and all-record totals for summary, category, account, transfer, and settlement metrics. Include source IDs, original card purchase/statement/credit event, recognized date, and signed contribution. Ensure each detail metric sums to its overview amount, including pages not initially loaded. Review query plans/indexes using 10,000-record fixture; add source-table indexes only if necessary.
6. **Complete backend gate**: Run feature/unit/contract tests for all spec acceptance cases, authentication, foreign IDs, invalid range/filter, period boundaries/leap cases, unequal-day percent suppression, zero/negative comparisons, pending/recurring/goal exclusions, archived history, refunds/corrections, and paged reconciliation. Run full suite and Pint inside development container; verify MySQL-specific recognition and report behavior. Frontend starts only when this gate passes.
7. **Frontend Reports**: Add protected route/nav and i18n; establish the canonical URL-backed scope and request guards with the US1 route/store foundation before detail and filter integrations. Later add controls for every period mode and filter without replacing that scope model. Compose summary, evolution, separate category sections, account table, comparison, filters, period selector, and report-specific contribution drawer. Reuse existing formatters and chart/theme wrapper with visible numbers/text alternatives.
8. **Frontend verification and outcomes**: Test URL restoration, rapid filter/period changes without mixed-context sections, no-data/no-income/no-expense/no-prior-data states, signed card adjustment detail, account movement labels, keyboard/screen-reader and color-independent meaning, 320 px/200% zoom, light/dark/system, PT-BR/English, and Playwright report-to-detail/filter/comparison journeys. Measure 10,000-record response/navigation target and uncoached SC-002/SC-003 tasks; record results for review.

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
└── checklists/requirements.md
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

**Structure Decision**: Follow current folder and JavaScript conventions. Exact filenames may be refined in task breakdown; shared card recognition stays in the existing card/dashboard/budget domain rather than a Reports-only override. No new library or reporting table.

## Frontend Component Map

| Component | Single responsibility | Inputs / outputs |
|---|---|---|
| `ReportsView` | Compose sections and coordinate one resolved scope | URL scope/store state → child props; handles child events |
| `ReportPeriodSelector` | Choose five quick period choices or navigate completed historical months | Scope in; period-change event out |
| `ReportFilterBar` | Choose account/category/type, show chips and reset | Filter options/active filters in; filter/reset events out |
| `ReportSummary` | Show realized income, expenses, and signed result | Summary/scope in; metric-detail event out |
| `ReportEvolution` | Present labelled interval trends with numeric alternative | Interval/granularity in; no domain calculation |
| `ReportCategoryBreakdown` | Present one classification's ranked categories and shares | Income or expense rows in; category-detail event out |
| `ReportAccountActivity` | Show account income/direct expense/net plus separate movements | Account rows in; metric-detail event out |
| `ReportComparison` | Show exact periods, absolute changes, and available percentages | Comparison/scope in; previous/current detail event out |
| `ReportContributionDrawer` | Show paged signed source records and reconciliation | Metric/scope/page in; request-next/close events out |

Pinia owns only state shared by route sections and detail. Pure formatting stays in existing utilities. A scope change invalidates stale overview/detail responses; computed values derive labels and visibility without client-side monetary recalculation. The report drawer handles card installments/credit events directly because the current Transactions detail click path skips `credit_card_expense` rows.

## Verification Gates and Risks

- **Money integrity**: Paid-source refund mismatch is a confirmed existing implementation gap. Fix and cross-feature regression coverage are prerequisites; a Report-only workaround would violate the spec.
- **Scope integrity**: One validated scope must cover all aggregates and drill-down; foreign IDs never reveal ownership; server returns effective dates/filters so stale client responses can be rejected.
- **Volume**: Keep overview aggregated and detail paged. Measure actual fixture performance before optional index work; no speculative caching.
- **Accessibility**: Charts supplement, never replace, labelled values and reachable records. Follow four design documents and existing Element Plus theme behavior.
- **Constitution**: No exceptions; the read repository is justified by complex cross-source aggregation and shared reconciliation.
