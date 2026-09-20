# Quickstart: Credit Cards Validation

## Prerequisites

- All repositories use branch `010-credit-cards`.
- Backend contract, authorization, validation, and tests land before frontend.
- Seed active/archived accounts, cards, categories; owner/foreign records;
  business dates around February/leap years; effective/pending/removed payments;
  paid and unpaid statements; source refunds/corrections.

## Backend validation

1. Create active card with R$ 5.000,00 limit, close 10/due 17. Confirm no
   credential field persists, current Open zero statement has exact period/due,
   and card starts used R$ 0,00/available R$ 5.000,00. Confirm archive rejects
   both outstanding obligation and positive card credit, then permits only zero/
   zero card.
2. Allocate purchase on closing day; verify it belongs to closing statement.
   Verify close 25/due 5 yields next-month due, close/due 31 clamp in February,
   and later installments cross year/month boundaries correctly. Update billing
   days and confirm allocated statements stay unchanged while later purchase
   follows new schedule.
3. Record R$ 100,00 over three installments. Verify amounts total exactly
   R$ 100,00, earliest installment gets residual centavo, full R$ 100,00
   reserves card limit, and account balances remain unchanged.
4. Confirm unconfirmed over-limit purchase is rejected with warning; confirmed
   purchase succeeds; concurrent stale confirmation cannot over-reserve credit.
5. Close applicable statement and verify only effective net installment is
   recognized in its closing month. Budget/Dashboard/History each count it once;
   before closing, confirm current/future closing-month installment appears as
   Expected. Purchase total and statement payment never duplicate it.
6. Make pending then effective partial/full payments from active owner account.
   Verify only effective payment debits account, changes statement/card exactly
   once, replays same idempotency key, rejects same key/different request, and
   supports edit/remove/restore/account-change restatement. Confirm R$ 100,00
   settlement from R$ 50,00 account succeeds with R$ -50,00 balance.
7. Cancel/refund/correct open and closed purchases. Verify preserved originals,
   source sequence allocation, paid-statement residual card credit, automatic
   oldest-statement application, nonnegative outstanding, and no cash movement.
8. Reject foreign/archived card/account/category, invalid centavos/dates/count,
   duplicate payment, over-outstanding payment, archive with debt, and every
   invalid state transition without data disclosure. Confirm historical archived
   associations stay readable.
9. Measure list/statement/projection reads with 10,000 combined sources at
   ≤2 seconds; inspect query plan before any index addition.

## Frontend validation

1. Navigate to Credit Cards with keyboard and compact drawer. Create/edit/archive
   card; list/detail shows separate stated limit, used, card credit, available,
   current zero statement, and archive restriction.
2. Create single and uneven installment purchases using visible category/card
   selection and BRL amount input. Verify close/due explanation, warning
   confirmation, field errors, loading guard, retry idempotency, and durable
   result.
3. Open statement detail: verify gross/net, installments, status/due text,
   payment/credit histories. Complete pending/partial/full payment and
   cancellation/refund/correction flows with focus return and clear non-cash
   credit explanation.
4. Confirm month Budget realized/expected values, Dashboard separate obligation
   card, and Financial History card-expense entry contain installment once; card
   payment appears only in card payment history and cash balance changes.
5. Test loading, empty, error/retry, unauthorized, Light/Dark/System, 320px,
   200% zoom, reduced motion, 44px targets, keyboard, labels, alerts, and
   non-color status clues.

## Commands

```bash
# backend
php artisan test --filter=CreditCards
vendor/bin/pint --dirty --format=agent

# frontend
npm run test:unit -- --run src/**/credit-cards/** src/services/__tests__/creditCardService.spec.js
CI=1 npm run test:e2e -- e2e/credit-cards.spec.js
npm run build

# contract
npx @redocly/cli lint specs/010-credit-cards/contracts/credit-cards-api.yaml
```
