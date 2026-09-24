# Implementation Plan: Recurring Credit Card Purchases

**Branch**: `014-recurring-credit-card-purchases` | **Date**: 2026-09-24 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/014-recurring-credit-card-purchases/spec.md`

**Branch Coordination**: `014-recurring-credit-card-purchases` is active in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

## Summary

Extend the existing recurrence domain with a fixed account-or-card destination.
Card rules default to automatic single-payment purchases and may instead await
owner confirmation with one-occurrence amount, date, card, or category changes.
Reuse card purchase, statement, and recognition rules; preserve account-rule
behavior and prevent duplicate purchases across retries and concurrent actions.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13.17+; JavaScript/Vue 3.5.40

**Primary Dependencies**: Sanctum 4, Eloquent, Axios 1.19, Pinia 4, Element Plus 2.14.5, Tailwind 4.3, Vue Router 5.2

**Storage**: Existing relational data, integer BRL centavos, owner-scoped recurrence/card records; no reporting aggregates

**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.61, contract validation

**Target Platform**: Authenticated Zunera web application

**Project Type**: Full-stack feature across backend and frontend repositories

**Performance Goals**: 95% of list/detail views usable within 2 seconds with 1,000 owned rules; due processing has exactly-once financial effect under retries

**Constraints**: America/Sao_Paulo business dates, exact centavos, atomic and idempotent financial mutations, no issuer access or new packages

**Scale/Scope**: Existing 50-item pages, 1,000-rule acceptance load, automatic and confirmation-based single-payment card expenses; account rules unchanged

## Delivery Scope and Order

1. **Backend** (`../zunera-backend`): Define protected recurrence and occurrence
   contract, destination validation, source-linked card purchases, generation and
   confirmation services, atomic uniqueness, financial projections, and tests.
2. **Frontend** (`../zunera-frontend`): Consume the verified backend contract in
   existing recurrence management, card-linked occurrence review, service/store
   state, accessible controls, and unit/end-to-end tests.

Frontend implementation starts only after backend contract, authorization,
validation, and tests pass.

## Research and Design Decisions

- [Research](research.md) records why the existing recurrence rule, calendar,
  scheduler, card purchase service, and billing/recognition rules are reused.
  Card occurrences have a durable scheduled-date identity, while account
  occurrences continue using their existing transaction source.
- [Data model](data-model.md) defines the destination migration, occurrence
  snapshots and workflow, unique purchase source link, state transitions, and
  reporting invariants. Existing account rules backfill as account destination.
- [API contract](contracts/recurring-credit-card-api.yaml) extends the existing
  owner-scoped recurrence API with card fields and confirm/dismiss/retry actions;
  purchase responses expose a nullable recurrence source. Account calls retain
  their existing required fields and behavior; new rule fields are additive.
- [Quickstart](quickstart.md) gives the backend-first build sequence, validation
  commands, and acceptance walkthrough.

## Implementation Sequence

1. **Compatibility and persistence:** Migrate rule destination and mode, card
   occurrence identity/snapshots, and nullable unique purchase source. Preserve
   current account rows, card rows, and existing idempotency records.
2. **Backend rule contract:** Extend owner-scoped Form Requests, DTOs, services,
   controllers, filters, and Resources. Enforce immutable destination type,
   active owned associations, card expense-only rules, and automatic default.
3. **Backend occurrence processing:** Branch the current due service by
   destination. Use the existing schedule calculator and daily job; commit each
   card date and source-aware purchase atomically, with unique constraints and
   consistent lock order. Surface expected, awaiting_over_limit, failed, dismissed, and
   recorded workflow without new financial states.
4. **Backend financial integration:** Reuse existing card purchase allocation,
   over-limit confirmation, statement reconciliation, credit calculation,
   refunds/corrections, and installment recognition. Extend Budget, Dashboard,
   upcoming/recent activity, and history where needed so one installment is
   counted once and an unconfirmed schedule stays a forecast.
5. **Backend verification gate:** Prove owner isolation, validation, account
   compatibility, dates and closing boundaries, late closed/paid statement
   restatement, retry/concurrency uniqueness, lifecycle, over-limit, and report
   integrity. Check representative 1,000-rule query performance and the
   documented API before any frontend work. Unit and feature suites must both pass.
6. **Frontend integration:** Extend the existing recurrence form/list/detail and
   occurrence review using the verified API. Show card identity, mode, source,
   current status, single-occurrence overrides, and actionable conflict text.
   Replace the card-exclusion copy. Preserve responsive Light/Dark/System design,
   keyboard use, 200% zoom, and PT/EN labels.
7. **Frontend verification:** Cover service/store and component behavior, then
   end-to-end creation, due expectation, confirmation, over-limit, navigation,
  and regression journeys. Check browser list/detail usability with 1,000 owned
  rules and the specification's configuration-time and status-understanding
  targets with representative users.

### Financial invariants checked at each stage

- Card occurrence date is unique per rule, and a recorded occurrence has at
  most one linked purchase with one installment. A card rule never creates an
  account transaction.
- An expected/awaiting_over_limit/failed/dismissed occurrence has no card obligation,
  account movement, or budget expense. A recorded purchase replaces its
  projected upcoming item and never implies issuer verification.
- The installment is the sole Expected/Realized expense source in the existing
  statement closing period. Statement payment only settles obligation; transfer
  and refund/correction meanings remain unchanged.
- Automatic late processing uses scheduled purchase date. Confirmation uses
  the owner's validated actual date, no later than the current São Paulo
  business date. Existing card billing and reconciliation
  decide statement allocation and restatement without repeating payment.

## Constitution Check

*GATE: Passed before research and re-checked after Phase 1 design.*

- [x] Protected routes, Form Requests, Resources, and owner-scoped services are
      required for changed actions; no upload or new secret is in scope.
- [x] Backend contract, authorization, validation, and tests precede frontend;
      frontend consumes only that confirmed contract.
- [x] Branch `014-recurring-credit-card-purchases` matches across all three repos.
- [x] Recurrence/card services own financial rules; multi-field rule/confirmation
      input warrants DTOs, while a new repository is not yet justified.
- [x] Frontend follows Composition API, Axios services, Pinia shared state, and
      Element Plus controls with established design tokens.
- [x] Backend unit/feature/contract and frontend unit/isolated Playwright journeys
      cover both modes, financial effects, concurrency, and compatibility.
- [x] Owner isolation, idempotency, exact centavos, no issuer integration, no
      credentials/uploads/secrets, and no package additions bound scope.
- [x] No constitution exception is proposed.
- [x] Phase 1 artifacts preserve the same service and validation boundaries,
      API compatibility, backend-first gate, test coverage, and no-package scope.

## Project Structure

### Documentation (this feature)

```text
specs/014-recurring-credit-card-purchases/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/
    └── recurring-credit-card-api.yaml
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/
│   ├── Data/RecurringTransactions/
│   ├── Http/Controllers/Api/V1/RecurringTransactionController.php
│   ├── Http/Requests/RecurringTransactions/
│   ├── Http/Resources/RecurringTransactions/
│   ├── Models/RecurringTransaction.php
│   ├── Models/CreditCardPurchase.php
│   ├── Services/RecurringTransactions/
│   ├── Services/CreditCards/
│   ├── Services/Budgets/
│   ├── Services/FinancialDashboard/
│   └── Services/FinancialHistory/
├── database/migrations/
├── routes/api.php
├── routes/console.php
└── tests/{Feature,Unit}/

../zunera-frontend/
├── src/
│   ├── views/recurring-transactions/RecurringTransactionsListView.vue
│   ├── components/recurring-transactions/
│   ├── services/recurringTransactionService.js
│   ├── stores/recurring-transactions/recurringTransactionStore.js
│   ├── utils/credit-cards/creditCardFormatters.js
│   ├── i18n/messages.js
│   └── router/
└── e2e/{recurring-transactions,credit-cards}.spec.js
```

**Structure Decision**: Backend work in `../zunera-backend` precedes frontend
work in `../zunera-frontend`; preserve the existing module layout. Frontend
work follows `docs/design/{design-foundation,app-shell,navigation,components}.md`
in this repository.

## Complexity Tracking

No constitution violations or new dependencies are proposed. The new card
occurrence identity is necessary to represent confirmation and retry states
without creating a financial purchase prematurely.
