# Implementation Plan: Transactions Month Navigator

**Branch**: `011-transactions-month-navigator` | **Date**: 2026-09-22 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/011-transactions-month-navigator/spec.md`

**Branch Coordination**: `011-transactions-month-navigator` is active in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

## Summary

Move the budgets month navigator into one shared frontend control and place it on
the transactions page's filter row, beside the search input and filter buttons.
The transactions page becomes month-scoped: it opens on the current business
month, the arrows move one inclusive calendar month at a time, the address keeps
the period, and clearing criteria keeps the month. Because the realized
income/expense/result cards read `meta.totals` from
`GET /api/v1/financial-history`, that request's totals become period-scoped for
any request that already carries `from` and `to`, while a request without a
period keeps today's all-time behavior. Backend contract, service change, and
tests land before the frontend consumes them.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13.17+; JavaScript/Vue 3.5.40
**Primary Dependencies**: Sanctum 4, Eloquent, Axios 1.19, Pinia 4, Element Plus
2.14.5, Tailwind 4.3, Vue Router 5.2, Vue I18n 11.4
**Storage**: No schema change; totals stay derived from existing owner-scoped
transactions
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.61
**Target Platform**: Authenticated Zunera web application
**Project Type**: Full-stack web feature
**Performance Goals**: Month-scoped list and totals load within 2 seconds for
10,000 owned movements using the existing count, union, and aggregate queries
**Constraints**: America/Sao_Paulo business month; inclusive first and last day;
existing 50-item page limit; no new package, migration, or stored aggregate
**Scale/Scope**: One owner per account; transactions filter row plus the shared
navigator adopted by the budgets page; removed-transactions screen and dashboard
period presets are out of scope

## Delivery Scope and Order

1. **Backend** — `GET /api/v1/financial-history` keeps its shape and scopes
   `meta.totals` to the requested period; Form Request validation, ownership, and
   pagination are unchanged; `FinancialHistoryService` owns the aggregate
   boundary; PHPUnit feature coverage lands with it in `../zunera-backend`.
2. **Frontend** — Shared `MonthNavigator` component and shared month utilities,
   transactions filter-row layout, view-level month scope with address sync,
   criterion and clear behavior, plus Vitest and Playwright coverage in
   `../zunera-frontend`.

Frontend implementation MUST start only after the relevant backend API contract,
authorization, validation, and tests are complete.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] The existing Sanctum-protected route, `ListFinancialHistoryRequest`
      validation, and owner scope already bound the endpoint; this feature adds no
      new route, upload, or secret, and keeps the controller thin.
- [x] Backend totals scoping and its tests land before the frontend filter row and
      month state consume the period-scoped response.
- [x] Branch `011-transactions-month-navigator` matches in specs, backend, and
      frontend.
- [x] The period aggregate stays in `App\Services\FinancialHistory`; no new DTO or
      repository is justified because the existing filter data object already
      carries the period.
- [x] Frontend uses `<script setup>`, a shared composition component, service
      calls through the existing Pinia store, Element Plus controls, and scoped
      styles that reuse existing semantic tokens.
- [x] Coverage is planned for every changed behavior: period totals, frontend unit
      tests for the navigator, filter bar, and view, and an isolated Playwright
      journey for month navigation.
- [x] Security, environment, and scope constraints are recorded; no new
      configuration is needed.
- [x] No constitution exception or new package is required.

## Project Structure

### Documentation (this feature)

```text
specs/011-transactions-month-navigator/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command)
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/
│   ├── Http/Controllers/Api/V1/FinancialHistoryController.php
│   ├── Http/Requests/FinancialHistory/ListFinancialHistoryRequest.php
│   └── Services/FinancialHistory/FinancialHistoryService.php
└── tests/Feature/FinancialHistory/

../zunera-frontend/
├── src/
│   ├── components/
│   │   ├── common/MonthNavigator.vue
│   │   ├── budgets/BudgetMonthNavigator.vue   (removed, consumers updated)
│   │   └── transactions/TransactionFilterBar.vue
│   ├── utils/common/monthFormatters.js
│   ├── views/transactions/TransactionsListView.vue
│   └── i18n/messages.js
└── e2e/transactions.spec.js
```

**Structure Decision**: The backend change is confined to the existing
financial-history controller and service; the frontend change promotes the
budgets navigator into `components/common`, adds month range helpers beside the
existing shared utilities, and keeps the transactions view as the owner of the
selected month.

## Complexity Tracking

No constitution violations; no complexity exceptions required.
