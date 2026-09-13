# Implementation Plan: Transaction Management

**Branch**: `004-transaction-management` | **Date**: 2026-09-11 | **Spec**:
[spec.md](spec.md)
**Input**: Feature specification from
`specs/004-transaction-management/spec.md`

**Branch Coordination**: `004-transaction-management` is checked out in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

**Note**: Planning ends after Phase 1 design. Task generation is the
responsibility of `/speckit-tasks`.

## Summary

Build authenticated income and expense transaction management on top of the
existing financial-account and category domains. The backend first delivers a
protected transaction contract with ownership-safe validation, account and
category association rules, pending/effective financial state, balance
consistency against each account's stored current balance, remove/restore
lifecycle behavior, filtering and search, and tests. The frontend then consumes
that contract through the authenticated app shell with a filterable
newest-first history, a record/edit form, a detail view, and removed-transaction
recovery, following the Zunera design foundation.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13.17+; JavaScript with Vue 3.5.40
**Primary Dependencies**: Laravel Sanctum 4, Form Requests, API Resources,
Eloquent; Axios 1.19, Pinia 4.0.2, Vue Router 5.2, Element Plus 2.14.5,
vue-i18n 11.4.8, Tailwind CSS 4.3, Vite 8.1
**Storage**: Existing relational application persistence. Transaction amounts
are stored as positive integer centavos; each transaction stores its own
financial status and removal state, and each financial account keeps its
existing materialized `current_balance_centavos`
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.62, Redocly
OpenAPI lint
**Target Platform**: Authenticated Zunera web application
**Project Type**: Full-stack web feature across specs, backend API, and
frontend SPA
**Performance Goals**: The first history batch of up to 50 entries renders
within 2 seconds for a user with 5,000 transactions; filtering, searching, and
saving a transaction behave like other authenticated CRUD screens
**Constraints**: No new packages; protected routes; service-owned business
rules; integer centavos only, never floating point; account balances must stay
consistent with the account's initial balance and effective movements;
archived accounts and categories remain valid historical associations but are
never selectable for a new association; removal is a lifecycle change, not
permanent deletion
**Scale/Scope**: At least 5,000 transactions per user; income and expense types
only; individual transactions only, with no transfers, recurring rules,
installments, invoices, budgets, goals, imports, or advanced reports

## Delivery Scope and Order

Document both scopes in this exact order:

1. **Backend** — Transaction API contract, persistence, authorization,
   validation, service-layer type/status/association rules, balance
   consistency, remove/restore lifecycle, filtering and search, and backend
   tests in `../zunera-backend`.
2. **Frontend** — Vue transaction history, form, detail, removed-transactions
   recovery, Axios service, Pinia state, Element Plus controls, design-compliant
   presentation, and frontend tests in `../zunera-frontend`.

Frontend implementation MUST start only after the relevant backend API contract,
authorization, validation, and tests are complete.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] API boundaries use `auth:sanctum` plus `session.lifetime` middleware, Form
      Requests for input, API Resources for output, and no upload handling.
- [x] Backend contract, authorization, validation, and tests precede frontend
      work in `../zunera-frontend`.
- [x] Branch name matches across all three repositories:
      `004-transaction-management`.
- [x] Transaction business rules live in focused services. Create/update/
      filter payloads justify DTOs; balance recalculation is service-owned;
      plain Eloquent access does not justify a repository.
- [x] Frontend uses JavaScript `<script setup>`, Axios service access, Pinia
      only for cross-view transaction state, and Element Plus standard controls.
- [x] Backend behavior and API-contract coverage plus isolated Playwright
      critical journeys are planned for every changed behavior.
- [x] Ownership denial, privacy-safe feedback, no secrets, no uploads, and
      explicit scope limits are recorded.
- [x] No constitution exception is required.

## Project Structure

### Documentation (this feature)

```text
specs/004-transaction-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/requirements.md
├── contracts/transactions-api.yaml
└── tasks.md              # Created by /speckit-tasks, not this command
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/
│   ├── Data/Transactions/
│   │   ├── CreateTransactionData.php
│   │   ├── UpdateTransactionData.php
│   │   └── TransactionFilterData.php
│   ├── Enums/Transactions/
│   │   ├── TransactionType.php
│   │   └── TransactionStatus.php
│   ├── Exceptions/Transactions/
│   │   └── TransactionStateException.php
│   ├── Http/
│   │   ├── Controllers/Api/V1/TransactionController.php
│   │   ├── Requests/Transactions/
│   │   │   ├── ListTransactionsRequest.php
│   │   │   ├── StoreTransactionRequest.php
│   │   │   ├── UpdateTransactionRequest.php
│   │   │   └── LifecycleTransactionRequest.php
│   │   └── Resources/Transactions/TransactionResource.php
│   ├── Models/Transaction.php
│   └── Services/Transactions/
│       ├── TransactionService.php
│       ├── TransactionIdempotencyService.php
│       ├── TransactionMoney.php
│       ├── TransactionDateRange.php
│       ├── TransactionTextNormalizer.php
│       └── TransactionBalanceReconciler.php
├── database/
│   ├── factories/TransactionFactory.php
│   └── migrations/
│       ├── *_create_transactions_table.php
│       └── *_create_transaction_mutation_requests_table.php
├── routes/api.php
└── tests/
    ├── Feature/Transactions/
    └── Unit/Transactions/

../zunera-frontend/
├── e2e/transactions.spec.js
├── src/
│   ├── components/transactions/
│   ├── router/index.js
│   ├── services/transactionService.js
│   ├── stores/transactions/transactionStore.js
│   ├── utils/transactions/transactionOptions.js
│   ├── utils/transactions/transactionFormatters.js
│   └── views/transactions/
└── tests through colocated `__tests__/` folders
```

**Structure Decision**: Follow the completed financial-account and category
vertical slices. Backend files stay grouped by domain, reuse the existing
integer-centavos money convention, and establish the contract before frontend
work. Frontend route views compose a filter bar, a history list with
progressive loading, a record/edit form, a details drawer, and confirmation
dialogs; transport behavior stays in the service and cross-view transaction
state stays in the feature store.

## Phase 0 Research Decisions

- Store amounts as positive integer centavos (`amount_centavos`) from
  R$ 0.01 (1) to R$ 999,999,999.99 (99,999,999,999). Integer storage preserves
  Brazilian currency precision exactly; a floating-point or decimal-string
  representation was rejected because it risks rounding drift in balances.
- Keep each account's existing materialized `current_balance_centavos` as the
  balance of record and reconcile it inside the same database transaction that
  changes a transaction. Deriving balances on read was rejected because the
  financial-account contract, resource, and summary endpoint already expose the
  stored balance.
- Reconcile by delta: an effective, non-removed income adds its amount and an
  effective, non-removed expense subtracts it. Every create, update, status
  change, account or type change, removal, restore, and account archive path
  computes the previous and resulting effect and applies the difference exactly
  once, locking the affected account row for the duration.
- Model financial state as two enum values, `pending` and `effective`. Only
  `effective` transactions contribute to a balance. Status may move freely
  between the two states, and each transition applies or removes the effect
  once.
- Default a new transaction to `effective`, except when its transaction date is
  in the future. A create request that asks for `effective` on a future date is
  rejected with a field-level validation message explaining that future-dated
  transactions start as pending; an explicit later update may still mark it
  effective.
- Model removal as a nullable `removed_at` timestamp rather than deletion. A
  removed transaction keeps its data, is excluded from balances and normal
  lists, appears in a dedicated removed view, and is restored by clearing
  `removed_at` and choosing `pending`/`effective` under the same date rules.
  Permanent deletion was rejected because it would break historical integrity.
- Persist the transaction type as an enum with `income` and `expense` values.
  The enum keeps the movement model extensible: a future transfer type is added
  as a new value with its own rules rather than by reinterpreting an existing
  row.
- Validate account and category ownership in the service, returning a
  privacy-safe 404 for another user's resources, matching the existing
  financial-account and category features. A user's own archived account or
  category is rejected for a new or changed association with a 422 field error
  instead, because existence is not a privacy concern.
- Allow an existing archived account or category association to be kept while
  other fields change. The archived association is rendered as a read-only
  choice; selecting a different archived account or category is rejected.
- Embed a minimal account summary and category summary inside the transaction
  resource, including their status. This keeps history, detail, and filter
  results understandable when an account or category has since been archived,
  without extra requests.
- Paginate the history server-side with a default and maximum page size of 50
  entries, newest first by transaction date and then by creation order, and
  return the total count of matching transactions. Progressive loading and the
  visible matching count satisfy the clarified scale rules.
- Implement filtering by date range, type, account, category, and status, and
  free-text search across description and notes with case- and
  accent-insensitive matching. Search text stays separate from structured
  filters so results remain predictable, and it is matched against the derived
  `search_text` column produced by `TransactionTextNormalizer` so behavior is
  identical on MySQL and SQLite.
- Require a fresh `Idempotency-Key` for every transaction mutation. Persist the
  owner-scoped key, request fingerprint, and completed response in the same
  database transaction as the mutation: matching retries replay the response;
  reused keys with a different request return typed 409 and never change a
  balance. This preserves legitimate identical transactions with different
  keys.
- Set the existing forward-compatible history flags when a transaction is
  created: `financial_accounts.has_financial_movements` and
  `categories.has_financial_transactions` become true and stay true, so the
  locks promised by those features take effect with real data instead of
  fixtures.
- Accept transaction dates as ISO `YYYY-MM-DD` or Brazilian `DD/MM/YYYY` input
  and normalize to a stored date value, restricted to 1900-01-01 through
  2100-12-31. Brazilian date input was kept because the product is
  Brazilian-currency and Brazilian-locale oriented.
- Use versioned authenticated JSON resource routes and the existing
  `{ "data": ... }` envelope, adding pagination metadata only on the paginated
  history endpoint.

## Phase 1 Design Outputs

- Research decisions: [research.md](research.md)
- Data model: [data-model.md](data-model.md)
- API contract: [contracts/transactions-api.yaml](contracts/transactions-api.yaml)
- Setup and verification: [quickstart.md](quickstart.md)
- Agent context: [AGENTS.md](../../AGENTS.md) now points to this plan.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |
