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
- Attempt to mark a pending future transfer effective, and to restore one as
  effective; verify each returns `422 effective_future_date` without balance
  change.
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

## Verification evidence (2026-09-13)

Backend commands ran inside the project PHP 8.3 container (`docker exec
zunera-backend-app-1`), because the host PHP lacks the SQLite driver that
`phpunit.xml` selects. The container has no `git`, so Pint ran on the feature
paths instead of `--dirty`; a dry run over `app tests routes` reported only
feature files, and those are now clean.

| Check | Command | Result |
|---|---|---|
| Backend feature/unit | `php artisan test --compact` | 160 passed, 1,006 assertions |
| Backend focused | `php artisan test --compact tests/Feature/Transfers tests/Unit/Transfers tests/Feature/FinancialHistory` | 49 passed, 382 assertions |
| Backend style | `vendor/bin/pint --format agent app/Services/FinancialHistory tests/Feature/FinancialHistory tests/Feature/Transfers tests/Unit/Transfers` | Fixed, then clean |
| Contracts | `npx @redocly/cli@latest lint contracts/transfers-api.yaml contracts/financial-history-api.yaml` | Both descriptions valid |
| Frontend unit | `npx vitest run` | 52 files, 159 tests passed |
| Frontend build | `npm run build` | Built successfully |
| Isolated journeys | `CI=1 npx playwright test e2e/transfers.spec.js` | 18 passed (chromium, firefox, webkit) |

Seeded 5,000-transfer first-batch evidence: `php artisan test --compact
tests/Feature/Transfers/TransferHistoryScaleTest.php` measured **13.0 ms** for the
first 50-item batch (newest first, exact total, progressive reachability, stable
same-date ordering) against the documented two-second budget of 2,000 ms.

## Final implementation notes

- `TransferStateException` carries both the conflict status and the
  `effective_future_date` 422 mapping, so pending-while-future and future-restore
  rules return the code the contract documents without a second exception type.
- `TransferBalanceReconciler` returns the proposed balances it validated, keyed in
  ascending account order, which is also the order it locks the accounts in.
- `GET /financial-history` adds an `income_centavos`/`expense_centavos`/
  `financial_result_centavos` totals block to its `meta`. The contracts allow extra
  metadata, and this is the reporting surface the existing history screen uses to
  prove transfers never enter income or expense totals.
- The frontend keeps `TransferRowActions.vue` separate from the view, matching the
  existing transactions slice and avoiding element-plus dropdown recursion inside
  a table cell.
- The transaction store normalizes mixed-history entries: income/expense entries
  regain `type`/`transaction_date`, and transfer entries keep their discriminator
  plus both account sides without being refetched as transactions.
