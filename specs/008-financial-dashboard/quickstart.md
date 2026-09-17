# Quickstart: Financial Dashboard Validation

## Prerequisites

- All repositories use branch `008-financial-dashboard`.
- Backend contract, authorization, validation, and tests land before frontend.
- Seed active/archived accounts; income/expense categories; effective/pending/
  removed transactions; effective/pending transfers; active recurrence.

## Backend validation

1. Confirm every route requires owner authentication; foreign data is
   indistinguishable from absence.
2. For inclusive period, verify only effective non-removed income/expense enters
   totals; exact result equals income minus expenses; transfers contribute zero.
3. Verify active balance excludes archived accounts while valid historical account/
   category labels remain understandable.
4. Verify custom/current/previous periods and daily/weekly/monthly buckets,
   including marked partial boundary interval.
5. Verify future pending transaction plus all eligible recurrence dates within 30
   days; generated occurrence appears once; past/current pending is Recent only.
6. Validate 10,000 owner movements within 2 seconds on initial/period reads.

## Frontend validation

1. Open `/app`: Dashboard title, current-balance label, current-month range.
   Change preset/custom dates; period sections update, current balance stays current.
2. Verify text/icon/sign labels for income, expense, transfer, pending, expected,
   and positive/zero/negative result; no color-only meaning.
3. Verify each section independently loads, empties with helpful CTA, errors safely
   with retry, while reliable sections remain usable.
4. Verify chart legend/value/table/text alternative; themes, 320px, 200% zoom,
   keyboard, reduced motion, and accessible labels.
5. Verify Recent Activity shows at most ten newest eligible records (exactly ten
   when more exist) and its full Financial History link preserves access to
   older records.

## Commands

```bash
# backend
php artisan test --filter=FinancialDashboard
vendor/bin/pint --dirty --format=agent

# frontend
npm run test:unit -- --run src/**/dashboard/**
CI=1 npm run test:e2e -- e2e/financial-dashboard.spec.js
npm run build

# contract
npx @redocly/cli lint specs/008-financial-dashboard/contracts/financial-dashboard-api.yaml
```
