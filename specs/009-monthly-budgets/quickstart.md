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
