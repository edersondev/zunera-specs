# Quickstart: Recurring Transactions Feature

## Backend first

1. Confirm `006-recurring-transactions` in `../zunera-backend`; add no package.
2. Implement and lint recurrence contract before frontend work.
3. Add rule/mutation persistence, source fields on transactions, enum/DTO,
   requests/resources/controller, owner-scoped service, and routes.
4. Implement calendar service using America/Sao_Paulo business date, creation
   eligibility anchor, weekly/monthly/yearly adjustment, inclusive end date,
   active/paused/ended transitions, and next expected date.
5. Implement due processor: lock eligible active rules, create each missing
   eligible occurrence pending in one transaction, and enforce source-rule/date
   uniqueness. It must catch up active downtime dates but never paused or
   pre-creation dates; mark expired rules ended.
6. Extend Transaction and FinancialHistory resources with nullable source link;
   provide the owner-scoped, paginated rule-occurrence collection that links to
   existing transaction detail; preserve transaction lifecycle/balance services.
7. On account/category archive, invoke recurrence service to pause affected
   active rules with association reason; archive never edits occurrences.
8. Add unit, feature, contract, duplicate/retry/concurrency, archive, and scale
   coverage before frontend gate.

## Frontend after backend gate

1. Confirm branch in `../zunera-frontend`; consume confirmed contract through
   recurrence Axios service and setup-style Pinia store.
2. Add authenticated Recurring Transactions navigation and routes: list/create,
   detail/edit, lifecycle actions. Use thin route view plus `RecurringTransactionFormDialog`,
   `RecurringTransactionFilterBar`, `RecurringTransactionList`, detail drawer,
   and lifecycle confirm dialog.
3. Reuse `CurrencyAmountInput`, active owned account/category lists, Element
   Plus forms/dialogs/drawers/tables, existing i18n patterns, and semantic tokens.
   Archived current association is a labelled repair state, never normal choice.
4. Add source-rule label/link in transaction detail and mixed history plus a
   paginated occurrence list in rule detail; individual occurrence edit stays
   transaction management.
5. Refresh recurrence list, history, and account state after rule/lifecycle
   action; show durable state plus accessible success/error/empty/loading feedback.
6. Verify themes, 320px/200% zoom, keyboard dialog/drawer focus, no-color-only
   source/state cues, filters/no-match, and critical journeys.

## Manual acceptance smoke test

- Create weekly salary and monthly 31st expense; verify expected date and
  February month-end adjustment.
- Create past-start rule; verify dates before creation are absent. Process due
  dates twice/concurrently; verify exactly one pending occurrence per date.
- Let processor resume after downtime; verify every eligible active missed date
  is pending and balance unchanged. Mark one effective; verify one balance effect.
- Pause/resume; verify paused dates never backfill. Archive association; verify
  automatic pause, repair requirement, unchanged history. End manually and by
  passed end date; verify no future occurrence.
- Edit rule amount then one occurrence; verify future ungenerated dates adopt
  rule value while prior/generated snapshots stay unchanged.
- Attempt cross-user access, inactive association, type/category mismatch,
  invalid amount/date, repeated lifecycle action, and changed idempotency-key
  reuse; verify safe rejection.

## Verification commands

    # Backend (use project PHP container when host lacks SQLite driver)
    php artisan migrate
    php artisan route:list -vv --path=api
    php artisan test --compact tests/Feature/RecurringTransactions tests/Unit/RecurringTransactions
    php artisan test --compact
    vendor/bin/pint --dirty --format agent

    # Recurrence and integrated history contracts
    npx @redocly/cli lint specs/006-recurring-transactions/contracts/recurring-transactions-api.yaml
    npx @redocly/cli lint specs/004-transaction-management/contracts/transactions-api.yaml
    npx @redocly/cli lint specs/005-account-transfers/contracts/financial-history-api.yaml

    # Frontend
    npm run test:unit -- --run
    npm run build
    CI=1 npm run test:e2e -- e2e/recurring-transactions.spec.js
