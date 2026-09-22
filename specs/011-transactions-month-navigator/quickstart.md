# Quickstart: Transactions Month Navigator Validation

## Prerequisites

- All repositories use branch `011-transactions-month-navigator`.
- Backend tests run inside the live Docker stack from `../zunera-backend`;
  frontend unit and browser tests run from `../zunera-frontend`.
- Backend contract, service change, and tests are complete before frontend work
  starts.
- Seed movements across two or more months, including first-day and last-day
  entries, an empty month, and pending/removed movements.

## Backend validation

1. Request `/api/v1/financial-history?from=2026-09-01&to=2026-09-30` and confirm
   `meta.totals` counts only effective, non-removed income and expense
   transactions dated inside September, with transfers excluded.
2. Confirm the same request excludes movements dated 2026-08-31 and 2026-10-01,
   and includes movements dated exactly 2026-09-01 and 2026-09-30.
3. Request a month without qualifying movements and confirm all three totals are
   zero, then request the endpoint without `from`/`to` and confirm totals match the
   previous all-time figures.
4. Confirm an inverted range (`from` after `to`) is still rejected, and that
   another user's movements never contribute to totals.

## Frontend validation

1. Open the transactions page with no query parameters: the header shows the navigator
   between the title and the "Nova transação" actions, it displays the current business
   month, and the list is limited to it.
2. Use the next and previous arrows across a year boundary and confirm the label,
   the list, the totals, and the address all move together, and that the controls
   are disabled while loading.
3. Apply search, type, account, and category criteria, then change the month and
   confirm those criteria survive while the period changes.
4. Apply a custom date range in the filters dialog: the period criterion appears
   alongside the other criteria, the navigator label follows the range start, and
   clicking an arrow replaces the range with the whole chosen month.
5. Remove the period criterion and then run Clear filters: the period returns to
   the navigator month while the other criteria are cleared.
6. Confirm an account without movements still shows the "nothing recorded yet"
   empty state on first load, and that the budgets month navigator keeps its
   previous behavior, keyboard support, and 320px layout.

## Commands

```bash
docker exec zunera-backend-app-1 php artisan test --filter=FinancialHistory
docker exec zunera-backend-app-1 vendor/bin/pint --format=agent
npm run test:unit -- --run
npm run lint
npm run build
CI=1 npm run test:e2e -- e2e/transactions.spec.js e2e/budgets.spec.js
```

## Validation Evidence (2026-09-22)

- Backend: `php artisan test --filter=FinancialHistory` → 13 passed (121 assertions);
  `vendor/bin/pint --format=agent` → passed.
- Frontend unit: `npm run test:unit -- --run` → 115 files, 428 tests passed; `npm run
  lint` passed.
- Frontend end-to-end: `CI=1 npm run test:e2e -- e2e/transactions.spec.js
  e2e/budgets.spec.js` → 42 passed across chromium, firefox, and webkit, covering the
  default month on load, month-scoped list and card totals, the period criterion for a
  custom range, the address update on month change, and keyboard use at 320px.
- Baseline check: the stashed (pre-change) checkout reproduces the budgets
  `progressbar` strict-mode failures, confirming they came from the earlier budget
  overview redesign rather than this feature.
