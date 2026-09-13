# Quickstart: Account Transfers Feature

## Backend first

1. Confirm 005-account-transfers branch in zunera-backend.
2. Do not add packages. Implement/lint transfer and mixed-history contracts
   before frontend work.
3. Add transfer and mutation persistence, focused service, validation requests,
   resources, and owner-safe routes.
4. Reconcile source/destination in one transaction. Lock distinct accounts in
   ascending identifier order; calculate proposed balances before persisting;
   reject source overdraft or destination-range breach with no partial change.
5. Require idempotency key on every mutation; exact retries replay and changed
   reuse returns typed conflict.
6. Preserve archived associations, set financial-movement history flag, and
   expose labelled transfer rows through `GET /financial-history`. Keep the
   transaction-only `GET /transactions` route unchanged and do not register a
   second owner for it.
7. Verify FinancialHistoryService excludes transfers from income, expense, and
   financial-result totals; verify FinancialAccountService applies both account
   effects while the combined owned-account net-worth total stays unchanged.
8. Add unit, feature, contract, and concurrency coverage before frontend begins.

## Frontend after backend gate

1. Confirm 005-account-transfers branch in zunera-frontend.
2. Use confirmed contract through transfer Axios service and setup-style Pinia
   store. New idempotency key per logical mutation; reuse only exact retry.
3. Add authenticated Transfers route with filters/search, create/edit form,
   detail drawer, and Removed restore view. Extend existing history with labelled
   Source → Destination transfer rows.
4. New selections use active owned accounts; selected source is unavailable as
   destination. Retain archived current associations as read-only labels.
5. Refresh transfer history and account balances after mutation. Surface source
   and destination impacts with durable UI updates plus accessible feedback.
6. Verify themes, narrow/zoom layouts, keyboard dialogs/drawers, loading/empty/
   error/no-match states, and non-color classification.

## Manual acceptance smoke test

- Create R$ 1.000,00 effective transfer from R$ 5.000,00 to R$ 2.000,00;
  verify R$ 4.000,00/R$ 3.000,00 and unchanged combined balance.
- Attempt source overdraft; verify error and both balances unchanged.
- Create future transfer; verify pending/no effect. After its date is today or
  past and funds exist, make it effective; verify effects apply once.
- Retime an effective transfer into future; verify both balance effects remain,
  the effective-future notice appears, and an explicit pending status change
  reverses both effects once.
- Edit amount, sides, date, and status; verify old effects reverse and new
  effects apply atomically.
- Remove effective transfer then restore effective; verify both effects reverse
  and return exactly once.
- Archive associated account; verify readable/editable retained association but
  unavailable new choice.
- Filter/search/no-match and 5,000-record progressive history; under the
  documented seeded test environment, record first-batch elapsed time of no more
  than two seconds.
- Verify mixed financial history labels transfer beside income/expense;
  FinancialHistoryService totals exclude transfer amount and FinancialAccountService
  preserves combined owned-account net worth.
- Retry mutations with same key and changed payload; verify replay/conflict.

## Verification commands

    # Backend
    php artisan migrate
    php artisan route:list -vv --path=api
    php artisan test --compact tests/Feature/Transfers tests/Unit/Transfers
    php artisan test --compact
    vendor/bin/pint --dirty --format agent

    # Contracts
    npx @redocly/cli lint specs/005-account-transfers/contracts/transfers-api.yaml
    npx @redocly/cli lint specs/005-account-transfers/contracts/financial-history-api.yaml

    # Frontend
    npm run test:unit -- --run
    npm run build
    CI=1 npm run test:e2e -- e2e/transfers.spec.js
