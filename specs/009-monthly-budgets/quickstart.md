# Quickstart: Monthly Budgets Validation

## Prerequisites

- All repositories use branch `009-monthly-budgets`.
- Backend contract/authorization/validation/tests land before frontend.
- Seed active/archived system/personal expense categories, income category,
  effective/pending/removed expenses, income, transfers, and pending recurrence.

## Backend validation

1. Create empty September 2026 budget. Confirm no transaction/balance/recurrence
   change; R$ 0,00 money totals and Not applicable utilization.
2. Add Food R$ 1.000,00. Reject duplicate, income, archived, unavailable,
   foreign, zero, negative, malformed, over-precision, out-of-range input.
   Confirm later income reclassification of budgeted Food is rejected, even
   before Food has a transaction.
3. Confirm only effective non-removed Food expense in full September realizes;
   income/transfers/pending/removed do not. Effective future-dated September
   expense still counts.
4. Confirm current/future pending expenses are expected; recurrence definition
   without generated transaction is not. Past month shows no projection.
5. Verify exact arithmetic, 80/100/exceeded/projected states, unbudgeted/total.
6. Archive linked category: snapshot/read-only. Restore: mutable. Copy rejects
   occupied/same destination or archived source atomically.
7. Correct transaction amount/date/category/type/state/removal; next read
   restates. Foreign records remain unavailable. Validate 10,000 rows ≤2 seconds.

## Frontend validation

1. Navigate `/app/budgets`; keyboard month controls/title/navigation work.
2. Verify empty/loading/error/retry/validation; plan actions never invoke
   financial-movement UI.
3. Verify values/status/excess/budgeted-unbudgeted-total/projection text; color
   supplemental only. Verify archived/copy/past-month rules.
4. Check Light/Dark/System, 320px, 200% zoom, focus, labels, alerts, 44px targets.

## Commands

```bash
# backend
php artisan test --filter=Budgets
vendor/bin/pint --dirty --format=agent

# frontend
npm run test:unit -- --run src/**/budgets/**
CI=1 npm run test:e2e -- e2e/budgets.spec.js
npm run build

# contract
npx @redocly/cli lint specs/009-monthly-budgets/contracts/budgets-api.yaml
```

## Validation record

| Check | Command | Result |
|---|---|---|
| Backend suite (all features) | `php artisan test` (app container) | 308 passed, 2197 assertions |
| Budgets backend focus | `php artisan test --filter=Budgets` | 48 passed, 424 assertions |
| Backend style | `vendor/bin/pint app/... tests/...` | no remaining style issues |
| Frontend unit/component | `npx vitest run` | 87 files, 315 tests passed |
| Budgets frontend focus | `npx vitest run src/**/budgets src/services/__tests__/budgetService.spec.js` | 9 files, 39 tests passed |
| Budgets journeys | `CI=1 npx playwright test e2e/budgets.spec.js --retries=0` | 18 passed (chromium, firefox, webkit) |
| Full journey suite | `CI=1 npx playwright test --retries=0` | 141 passed (chromium, firefox, webkit) |
| Production build | `npm run build` | built, `BudgetsView` chunk emitted |
| Contract | `redocly lint contracts/budgets-api.yaml` | valid |
| Performance | `tests/Feature/Budgets/BudgetPerformanceTest.php` | 10,000 movements read in 0.51s; index-backed plan, no new index added |

Notes:

- `npm run lint` still fails on pre-existing findings outside this feature:
  oxlint in `src/composables/useChartTheme.js`, `e2e/transfers.spec.js`, and
  `e2e/recurring-transactions.spec.js`; eslint in
  `src/views/categories/{ArchivedCategoriesView,CategoriesListView}.vue` and
  `src/views/financial-accounts/FinancialAccountsListView.vue`. No budgets file
  reports a finding in either linter.
- Budget classification locking is stored as `categories.has_budget_plans`, set
  by the first plan association and never cleared, so removing a plan cannot
  unlock a reclassification.
