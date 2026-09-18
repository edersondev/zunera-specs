# Implementation Plan: Monthly Budgets

**Branch**: `009-monthly-budgets` | **Date**: 2026-09-18 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/009-monthly-budgets/spec.md`

## Summary

Deliver owner-scoped monthly expense budgets. Persisted monthly plans and their
category-plan snapshots hold planning inputs only. Realized, expected,
available, utilization, status, projected, budgeted, and unbudgeted values are
derived from authoritative transactions at read-time. Budget mutations never
call transaction or balance services. Backend contract, authorization,
validation, services, migrations, and tests complete before frontend consumes
the contract.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13.17+; JavaScript/Vue 3.5.40  
**Primary Dependencies**: Sanctum 4, Eloquent, Axios 1.19, Pinia 4, Element
Plus 2.14.5, Tailwind 4.3, Vue Router 5.2, Vue I18n 11.4  
**Storage**: Existing relational data plus monthly-budget/category-plan records;
integer BRL centavos; no persisted financial aggregates  
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.61, Redocly  
**Target Platform**: Authenticated Zunera web application  
**Project Type**: Full-stack web feature  
**Performance Goals**: Open or switch selected month within 2 seconds for up to
10,000 user financial movements  
**Constraints**: America/Sao_Paulo calendar-month boundaries; exact centavos;
realized from effective non-removed expenses only; expected from pending expense
transactions only for current/future months; no balance reconciliation from
budget code; accessible semantic status at 320px/200% zoom  
**Scale/Scope**: One user/month budget; unique category plan per budget;
historical snapshot identity; no dashboard presentation work in this feature.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] Sanctum, Form Requests, API Resources, owner-scoped services, no
  uploads/secrets protect new boundaries.
- [x] Backend contract, authorization, validation, services, migrations, and
  tests precede frontend work.
- [x] `009-monthly-budgets` is active in specs, backend, and frontend.
- [x] Focused lifecycle/calculation services and input DTOs are justified;
  repository abstraction is not.
- [x] Frontend uses JavaScript `<script setup>`, Axios, Pinia, Element Plus,
  Tailwind semantic tokens, and props/events.
- [x] Contract plus backend/frontend/Playwright coverage are planned.
- [x] Ownership, no money movement, centavos, snapshot history, accessibility,
  and exclusions are explicit. No exception/new package required.

## Project Structure

```text
specs/009-monthly-budgets/
├── {plan,research,data-model,quickstart}.md
└── contracts/budgets-api.yaml

../zunera-backend/
├── app/{Data,Enums,Exceptions,Http,Models,Services}/Budgets/
├── database/{factories,migrations}/
├── routes/api.php
└── tests/{Feature,Performance,Support,Unit}/Budgets/

../zunera-frontend/
├── src/components/budgets/
├── src/views/budgets/BudgetsView.vue
├── src/services/budgetService.js
├── src/stores/budgets/budgetStore.js
├── src/utils/budgets/budgetFormatters.js
├── src/{router/index.js,layouts/AppShell.vue,i18n/messages.js}
└── e2e/budgets.spec.js
```

**Structure Decision**: `BudgetService` owns mutations, copy safety, ownership,
category eligibility, and snapshots. `BudgetCalculationService` owns read-time
aggregates. Controllers only map validated DTOs to these services/resources.
`BudgetsView` composes focused props/events components; its Pinia store owns
month fetch/mutation state. Formatters only render supplied values.

## Delivery Scope and Order

1. **Backend** — migrations/models/factories; lifecycle/calculation services;
   DTOs/requests/resources/controller/routes; API contract; unit, feature, and
   performance tests. Prove no budget action mutates transactions/balances.
2. **Frontend** — after backend passes, add route/nav/i18n, Axios service, Pinia
   store, focused view/components, accessibility, and automated coverage.
3. **Dashboard boundary** — do not add dashboard widget now. Later Dashboard
   consumes Budget's compact derived summary; it never recomputes budget logic.

## Backend Design

### Persistence and lifecycle

- Add `MonthlyBudget`: owner, `budget_year`, `budget_month`, timestamps; unique
  owner/month and indexed lookup. Empty budgets are valid. Period never changes;
  v1 has no delete-month operation.
- Add `BudgetCategoryPlan`: parent, linked category, positive
  `planned_amount_centavos`, `currency_code` BRL, and immutable category snapshot
  (name, classification, origin, color, icon). Unique parent/category.
- New plan requires active available expense category. Archived linked category
  remains calculated/displayed from snapshot but is read-only; restoration
  enables normal mutation without replacing snapshot. First plan association
  also locks linked category classification as expense, even without a financial
  transaction. Plan removal does not unlock classification and affects only that
  plan, making matching expenses unbudgeted.

### Authoritative calculations

- Budget month is full first-to-last calendar month. Do not reuse dashboard's
  current-through-today period; stored effective future-dated transaction counts
  in its month.
- Realized: owned, non-removed, effective, expense transaction, same category,
  in selected month. Transfers do not join query.
- Expected: same owner/category/month, non-removed pending expense transaction
  only for current/future selected month. Generated pending recurrence qualifies;
  recurrence definition alone does not. Past month omits expected/projected.
- Projected = realized + expected. Actual available/utilization/status use
  realized. Thresholds server-owned: <80 within, 80–<100 approaching, 100
  reached, >100 exceeded. Empty budget returns zero money totals and null/
  Not-applicable utilization/status/progress.
- Summary derives planned, budgeted realized, available, overall utilization,
  unbudgeted effective expense, and total effective expense. Persist none of
  these. Add composite transaction index only if 10,000-movement measurement
  proves it needed.

### API and concurrency

- Contract: [budgets-api.yaml](contracts/budgets-api.yaml): month read/create,
  nested plan add/update/remove, source-budget copy.
- Month read returns `200` with `budget: null` for no-budget state.
- Mutations run in database transaction. Unique collision becomes clear conflict.
  Copy validates owner, different destination, empty destination, all source
  categories active before inserting any destination plan: all-or-nothing.
- Foreign data is indistinguishable from absent. Contract money is integer
  centavos after exact client BRL parsing—never float.

## Frontend Design

### Route and state

- Add authenticated lazy `/app/budgets` (`budgets`) route, primary navigation,
  title metadata, `app.budgets` and `budgets.*` translations in pt-BR/en.
- `budgetService` unwraps data and centralizes CSRF/error mapping. Setup-store
  uses `shallowRef` month data/loading/error and explicit fetch/create/plan/copy
  actions. Successful mutation refreshes only selected month.
- Selected `YYYY-MM` month remains route-local. Client does not calculate money,
  status, or financial totals; server resource is source. Formatters reuse or
  extract dashboard BRL formatting and only create locale labels.

### Component map

| Component | Responsibility | Props / emits |
|---|---|---|
| `BudgetsView` | Compose page/store/routing/dialogs | handles intents |
| `BudgetMonthNavigator` | Selected month, previous/next controls | month, loading / `change-month` |
| `BudgetSummary` | Derived summary + textual progress/status | summary, locale |
| `BudgetPlanList` / `BudgetPlanRow` | Plans, values, status, projections/actions | plans / `edit`, `remove`, `add` |
| `BudgetPlanFormDialog` | Add/edit positive expense plan | plan/categories/errors / `submit`, `close` |
| `BudgetEmptyState` | No-budget/no-plan/no-expense actions | state / `create`, `copy`, `add` |
| `CopyBudgetDialog` | Destination selection/conflict feedback | source / `copy`, `close` |
| `RemoveBudgetPlanDialog` | Plan-only removal confirmation | plan / `confirm`, `close` |

Use `ElForm label-position="top"`; map server field errors; reset form and
errors on dialog closure. Archived rows have archived text/tag and no edit/remove
controls. Progress always includes percent, status, values, and excess text.

### Responsive/accessibility

- Use shell gutters: 16px compact, 24px medium, 32px wide; single column under
  640px and scan-friendly summary/list above.
- Require one `h1`, named month controls, visible labels, `aria-live` feedback,
  semantic headings, focus-return dialogs, 44px targets, Light/Dark/System,
  keyboard, 200% zoom, 320px, and reduced motion support.

## Test Strategy

### Backend

- Unit: month DTO/bounds, centavos, eligibility, snapshot, thresholds,
  zero-plan Not-applicable, classification lock, source filters, copy
  preconditions, no-money action.
- Feature: auth/owner isolation; empty/create/add/update/remove; duplicate/race;
  archived read-only/restored mutable; atomic copy conflicts; validation/resource
  shape.
- Integrity: effective/pending/removed/income/transfer/re-categorized/re-dated/
  restored/future-effective transactions; generated pending recurrence; no
  ungenerated recurrence; unbudgeted/total; next-read restatement.
- Performance: 10,000 movements under 2 seconds; inspect query plan before index.

### Frontend

- Service/store: requests, CSRF/data unwrap, slices, month change, validation,
  failed mutation stability, duplicate-submit prevention.
- Formatter/components/view: BRL/month/Not-applicable, 80/100/>100 status,
  unbudgeted/projection labels, archived/read-only, empty/retry/dialog lifecycle.
- Playwright: authenticated nav/route, 320px keyboard, no financial-movement
  request on plan actions, realized exclusions, copy conflicts/archive, past
  projection omission, Light/Dark accessible text.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| N/A | N/A | N/A |
