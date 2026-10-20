# Quickstart: Manage Credit Card Purchases

## Scope and order

Use the `024-manage-card-purchases` branch in specs, backend, and frontend. The frontend lockfile refresh was committed separately; leave it outside feature implementation. Complete backend contract, authorization, validation, services, migrations, and focused tests before frontend work. No new package is needed.

## Backend verification path

1. Apply migrations in the existing backend environment. Confirm the move-history and payment-surplus tables, credit-application supersession field, and recurring purchase-deleted marker exist.
2. With an owned active card closing on day 10, request statement options for a purchase dated on day 10 and on day 9 after that cycle closes. Day 10 belongs to its own inclusive cycle; after closing, the form requires an explicit choice. Change the card's closing day to 25 while its January 10 statement remains stored; a January 15 purchase must resolve to the January 25 cycle, and both exact closing-date options must remain distinguishable. Jump the options query to an empty future month and verify its calculated closing/due dates without creating a statement row.
3. Create a backdated single-payment purchase with `statement_closing_date`, then one with installments. Verify the first selected cycle and later consecutive cycles, exact centavos, and unchanged cash-account balance. Omitted or foreign/invalid choice fails with field errors and no purchase.
4. Preview and move a full schedule. Move one installment so two share a statement, then run Full schedule again and verify consecutive rebuilding. Reject any individual move that reverses sequence order. Repeat a confirmed move with the same idempotency key and verify one history/effect. On a moved purchase whose current statements are open, PATCH description/category/notes and verify installment IDs, placement, recognition dates, and history persist; PATCH changed card/date/amount/count values and verify conflicts with no mutation.
5. Move a purchase from a paid statement. Verify original payment and paying-account debit remain, source payment surplus is applied to oldest unpaid statement, residual card credit is visible, and used/available credit and budgets/history reconcile to the centavo. Repeat with a linked refund/correction event and verify active credit applications follow the moved installment.
6. Delete a purchase without linked credits. Verify purchase, installments, and move history disappear; paid statement payment remains; credit and financial views update once. Verify identical delete replay returns the minimal receipt, new-key retry has no effect, and an old create/update replay cannot return deleted purchase data or recreate it.
7. Attempt to delete a purchase with any linked credit event and verify conflict with no data changes. Delete a purchase from a recorded recurring occurrence and verify the occurrence remains recorded with a deleted marker and cannot regenerate.
8. Run deterministic matrices: 100 distinct move allocations across month/year boundaries (SC-002), 100 distinct before/on/after-closing dates against both API and form (SC-003), and 100 distinct move/delete accounting states (SC-004). For SC-005, count 100 repeated or overlapping command attempts across varied starting states; each retry pair or overlap group must leave at most one financial effect. Assert the executed count for each matrix.
9. With a seeded card containing 1,000 purchases, warm up each statement-options and move-preview route 10 times, then measure 100 requests per route. Assert each route's p95 response time is under 2 seconds and inspect query plans for bounded cycle discovery. Complete this backend check before frontend work.

Preferred backend commands from `../zunera-backend/` in the live Docker stack:

```bash
rtk docker exec zunera-backend-app-1 php artisan migrate
rtk docker exec zunera-backend-app-1 php artisan test --filter=CreditCards
rtk docker exec zunera-backend-app-1 vendor/bin/pint --format=agent
```

Use the project-wide backend suite after focused financial and API-contract tests pass:

```bash
rtk docker exec zunera-backend-app-1 php artisan test
```

## Frontend verification path

1. Open New purchase on a card. A date in an open cycle shows no extra select. A date in a closed cycle shows an unselected Statement control; saving without a choice shows a field error. Choose a computed future cycle and verify the saved purchase appears there.
2. From purchase history, open Move, choose Full schedule, inspect the server preview, and confirm. Repeat One installment from a statement line; verify it preselects the clicked installment. Refresh both source and destination statements and move history.
3. Open Delete, dismiss once, then confirm. Check permanent-removal copy, retained payment display, updated card credit, and blocked linked-credit feedback.
4. Repeat at 320px and 200% zoom, keyboard only, in Light/Dark/System themes. Check labelled fields, focus return, readable currency/status text, and pt-BR/en messages.
5. Run a scripted check with 20 card-feature users on fixture data: five each for Full schedule/mobile, One installment/mobile, Full schedule/desktop, and One installment/desktop. Measure Move from opening to confirmation without facilitator help (at least 19 finish within 2 minutes). Before confirmation, ask each user which installments change and whether Delete is permanent; after confirmation, time discovery of the changed statement or totals (at least 18 answer correctly and find the result within 30 seconds). Record anonymous participant IDs, mode/layout, timings, and pass/fail in `usability-results.md`; do not record personal or financial data.

From `../zunera-frontend/`:

```bash
rtk npm run test:unit -- --run
rtk npm run lint
rtk npm run build
rtk proxy env CI=1 npm run test:e2e -- e2e/credit-cards.spec.js
```

The Playwright CI preview server uses the build from the prior command. Its isolated fixtures must cover the changed request/response contract, including stale preview, linked credits, and paid-statement surplus.

## Contract and financial acceptance

- Validate request/response shapes against [purchase-statement-management-api.yaml](contracts/purchase-statement-management-api.yaml) and the existing Credit Cards 010 contract.
- In every successful journey, gross installments retain their exact purchase total; statement outstanding is nonnegative; used credit equals nonnegative card obligations; residual card credit is separately visible; account cash moves only for existing effective payments.
- In every rejected or repeated journey, no partial purchase, duplicate installment, duplicated cash movement, or second recognized expense may remain.
