# Implementation Plan: Transactions Activity Redesign

**Branch**: `013-transactions-activity-redesign` | **Date**: 2026-09-23 | **Spec**: [spec.md](spec.md)
**Input**: Existing Spec 004 domain and approved frontend UI plan.

## Summary

Replace table/mobile split with date-grouped accordion. Route-backed month/custom date state drives existing financial-history endpoint. Existing Dashboard summary endpoint provides realized period values when no other filters apply. Keep all mutation flows and financial semantics.

## Technical Context

**Language/Version**: JavaScript, Vue 3.5 Composition API
**Primary Dependencies**: Vue Router, Pinia, Axios, Element Plus, Vue I18n, Tailwind 4
**Storage**: None; existing backend only
**Testing**: Vitest, Playwright, ESLint, Vite build
**Target Platform**: Responsive authenticated web app, light/dark/system
**Project Type**: Frontend-only feature
**Performance Goals**: One history request per period/filter change; no per-row detail requests until existing view-details action
**Constraints**: No API, database, package, or financial calculation change
**Scale/Scope**: Existing 50-entry load-more pages; grouping over loaded entries

## Delivery Scope and Order

1. **Backend**: No changes. `/api/v1/financial-history` supplies paginated mixed movements; dashboard summary accepts inclusive custom `from`/`to` and returns realized values. Existing authorization, validation, resources, and tests stay intact.
2. **Frontend**: Build shared month navigator, period coordination, grouped expandable history, period summary, and tests in `../zunera-frontend`.

## Constitution Check

- [x] API boundaries: existing authenticated endpoints and validation unchanged.
- [x] Backend contract already established before frontend work.
- [x] Specs/frontend branches match; backend branch omitted under AGENTS.md frontend-only rule.
- [x] No domain logic, DTO, repository, or persistence change.
- [x] Vue Composition API, services for transport, Pinia for existing shared state, Element Plus controls.
- [x] Vitest and Playwright cover changed behavior and critical journeys.
- [x] No secret, upload, package, or unrelated-page change.

## Project Structure

```text
specs/013-transactions-activity-redesign/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── contracts/history-ui.md
├── quickstart.md
└── tasks.md

../zunera-frontend/src/
├── components/common/MonthNavigator.vue
├── components/budgets/BudgetMonthNavigator.vue
├── components/transactions/TransactionHistoryList.vue
├── components/transactions/ExpandableHistoryItem.vue
├── components/transactions/TransactionFinancialSummary.vue
├── views/transactions/TransactionsListView.vue
└── utils/transactions/transactionPeriod.js
```

## Design

- URL `from`/`to` is period truth. On entry without either bound, replace URL with current São Paulo month. Full-month bounds select navigator month; non-full or open-ended bounds show custom badge. Month arrow writes new full-month bounds and resets page. Existing other filters are retained.
- Keep existing filter dialog and chips; compact toolbar styling. Use route watcher for external/back navigation and avoid duplicate fetch. Normalize/reset page in existing store `setFilters`; guard stale responses on rapid filter changes.
- Group loaded entries by `movement_date` without resorting; key each item by `movement_kind` + ID. Row header expands details only; existing details drawer remains a separate explicit action for supported types. Category icon resolves through active catalog when available, generic icon otherwise.
- Use dashboard summary custom preset for complete date bounds. Hide summary with any non-date filter, clear stale values while loading or on error, and note its realized-only exclusions. Never use history metadata lifetime totals as month totals.
- In empty unfiltered month, use an existing per-page-one history query only if needed to distinguish global absence from empty period. Filtered empty state needs no probe.

## Design Foundation

Follow `docs/design/design-foundation.md`, `app-shell.md`, `navigation.md`, and `components.md`: semantic tokens, 4px spacing scale, compact 44px targets, one h1, visible focus, no raw colors, and mobile-first layout. Keep Budgets navigator visual behavior and selectors.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
| --- | --- | --- |
| Backend branch omitted | Frontend-only task; AGENTS.md expressly scopes matching application branches to affected apps | Empty backend branch adds noise without code or contract change |
