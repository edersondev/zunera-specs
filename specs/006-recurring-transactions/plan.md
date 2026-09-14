# Implementation Plan: Recurring Transactions

**Branch**: `006-recurring-transactions` | **Date**: 2026-09-14 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/006-recurring-transactions/spec.md`

**Branch Coordination**: `006-recurring-transactions` is checked out in specs,
`../zunera-backend`, and `../zunera-frontend`.

## Summary

Deliver owned weekly, monthly, yearly income/expense templates. Each due rule
creates one pending ordinary transaction per eligible scheduled date; only an
owner later marking it effective affects a balance. Backend owns calendar,
catch-up, lifecycle, archive pauses, source links, paginated occurrence access,
and duplicate safety before frontend delivers accessible recurrence management
and labelled history.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13.17+; JavaScript/Vue 3.5.40  
**Primary Dependencies**: Sanctum 4, Eloquent, Axios 1.19, Pinia 4, Element Plus
2.14.5, Tailwind 4.3, Vue Router 5.2  
**Storage**: Existing relational persistence; integer BRL centavos; date-based
rules and source-linked existing transactions  
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.61, Redocly  
**Target Platform**: Authenticated Zunera web application  
**Project Type**: Full-stack web feature  
**Performance Goals**: Idempotent due processing; first 50 matching rules from
5,000 user rules in under 2 seconds in seeded validation.  
**Constraints**: No packages; BRL exact centavos; protected API; idempotent user
mutations; America/Sao_Paulo business date; no rule balance effect.  
**Scale/Scope**: At least 5,000 rules/user; income/expense only; no transfer,
installment, import, reminder, external billing.

## Delivery Scope and Order

1. **Backend** — API contract, authorization, Form Requests, DTOs, models,
   migrations, calendar/occurrence/lifecycle services, transaction integration,
   paginated occurrence access, archive hooks, due processing, and tests in
   `../zunera-backend`.
2. **Frontend** — Service, Pinia store, route/navigation, recurrence components,
   history-source display, translations, unit tests, and Playwright in
   `../zunera-frontend`.

Frontend starts only after backend contract, authorization, validation, and tests.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] Middleware, Form Requests, Resources, and no uploads secure API boundaries.
- [x] Backend contract/authorization/validation/tests precede frontend.
- [x] Branch names match in all three repositories.
- [x] Focused services own rules; multi-field inputs use DTOs; no repository needed.
- [x] Vue Composition API, Axios service, focused Pinia state, Element Plus, and
      props-down/events-up boundaries are planned.
- [x] Contract, backend unit/feature/concurrency, frontend unit, and isolated
      Playwright coverage cover changed behavior.
- [x] Ownership, idempotency, business date, no secrets/uploads, and scope are recorded.
- [x] No constitution exception required.

## Project Structure

### Documentation (this feature)

```text
specs/006-recurring-transactions/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/requirements.md
└── contracts/recurring-transactions-api.yaml
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/{Data,Enums,Exceptions,Services}/RecurringTransactions/
├── app/Http/{Controllers/Api/V1,Requests,Resources}/RecurringTransactions/
├── app/Models/{RecurringTransaction,Transaction}.php
├── app/Console/Commands/
├── database/{factories,migrations}/
├── routes/api.php
└── tests/{Feature,Unit}/RecurringTransactions/

../zunera-frontend/
├── e2e/recurring-transactions.spec.js
├── src/components/recurring-transactions/
├── src/views/recurring-transactions/
├── src/services/recurringTransactionService.js
├── src/stores/recurring-transactions/recurringTransactionStore.js
├── src/utils/recurring-transactions/
├── src/{router/index.js,layouts/AppShell.vue,i18n/messages.js}
└── src/components/transactions/TransactionDetailDrawer.vue
```

**Structure Decision**: Recurrence is one backend vertical slice. Its occurrence
service creates source-linked existing transactions, exposes their bounded
paginated list, and uses existing transaction balance reconciliation; no second
effect model. Frontend route remains thin and composes form, filter bar, list,
detail drawer, and lifecycle dialog.

## Phase 0 Research Decisions

- Calendar, catch-up, source link, uniqueness, archive pause, lifecycle,
  idempotency, and UI boundaries: [research.md](research.md).

## Phase 1 Design Outputs

- [data-model.md](data-model.md)
- [contracts/recurring-transactions-api.yaml](contracts/recurring-transactions-api.yaml)
- Existing transaction and financial-history contracts gain the nullable
  `recurrence_source` representation used by their established endpoints.
- [quickstart.md](quickstart.md)
- [AGENTS.md](../../AGENTS.md) points to this plan.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |
