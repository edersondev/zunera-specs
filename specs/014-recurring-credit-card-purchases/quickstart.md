# Quickstart: Recurring Credit Card Purchases

## Read before implementation

1. Use branch `014-recurring-credit-card-purchases` in this repository and in both sibling application repositories; verify each local branch before editing it.
2. Read [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), and [the API contract](contracts/recurring-credit-card-api.yaml). The clarified business decisions are authoritative.
3. Follow the existing feature architecture and the design references: `docs/design/design-foundation.md`, `docs/design/app-shell.md`, `docs/design/navigation.md`, and `docs/design/components.md`.
4. Deliver backend migration, authorization, validation, API, purchase integration, and tests first. Start frontend implementation after the backend contract and tests are verified. No new package is required.

## Backend path

1. Extend recurrence persistence without changing existing account-rule meaning. Add durable card occurrence identity and nullable unique source link on card purchases. Verify fresh and upgraded databases.
2. Extend Form Requests, DTOs, owner-scoped services, and Resources for destination and mode. Preserve old account request/response behavior.
3. Branch existing due processing by destination. For card rules, replace the whole-rule backlog transaction with one commit per scheduled date. Reserve each date once; confirmation creates an expected item; automatic generation goes through the current card purchase rules. Persist a recoverable failed identity after purchase rollback and continue later dates. If failure identity cannot be stored, stop that rule before advancing its cursor. Keep awaiting-over-limit/failed dates actionable and visible.
4. Add owner-scoped confirmation, dismissal, and failed-automatic retry actions, with idempotency and the existing over-limit conflict. Claim one confirmation attempt and its chosen values before purchase creation; reject competing distinct actions, then release or safely recover interrupted claims. Before an owner-initiated card rule edit, pause, or end, represent already-due dates under the old definition; reject the mutation if this cannot be done. Keep card/category archival from rewriting represented dates.
5. Reconcile statement, limit, budget, Dashboard, history, upcoming, and recent projections from the single purchase/installment source. Include source links in card activity.
6. Add feature and unit coverage for exactly-once behavior, owner isolation, account compatibility, calendar and statement boundaries, backdated closed/paid statements, over-limit, and correction/refund.

Preferred backend checks from `../zunera-backend/` (running Docker stack):

```bash
docker exec zunera-backend-app-1 php artisan test --filter=Recurring
docker exec zunera-backend-app-1 php artisan test --filter=CreditCard
docker exec zunera-backend-app-1 php artisan test --filter=Budget
docker exec zunera-backend-app-1 php artisan test --filter=Dashboard
docker exec zunera-backend-app-1 php artisan test
docker exec zunera-backend-app-1 vendor/bin/pint --format=agent
```

The sandbox may require escalation for Docker socket access. Validate the API shape against `contracts/recurring-credit-card-api.yaml` before frontend work.

## Frontend path

1. Extend the current Recurring Transactions form, list, filters, detail drawer, service, and Pinia store. Show an active card selector for card destinations; lock the destination type on edit; show mode and recurring-versus-installment guidance.
2. Give expected, awaiting approval, failed, dismissed, and recorded card occurrences distinct text and actions. Let confirmation edit one amount/date/card/category; after failure, display and reuse the last validated choices unless the owner changes them. Use the recurrence endpoint for confirmation, fresh idempotency keys on over-limit approval, and the existing card identity formatter.
3. Link recorded occurrences to the purchase and its statement activity. Suppress the unsupported-card message and update Portuguese/English copy. Check keyboard focus, narrow layouts, 200% zoom, and Light/Dark/System themes.
4. Update service/store/component tests and critical Playwright journeys, including the previous card exclusion assertion.

Frontend checks from `../zunera-frontend/`:

```bash
npm run test:unit -- --run
npm run lint
npm run build
CI=1 npm run test:e2e -- e2e/recurring-transactions.spec.js
CI=1 npm run test:e2e -- e2e/credit-cards.spec.js
```

## Acceptance walkthrough

1. Create an automatic R$ 150 monthly gym expense for day 12 on an active card. Run due processing twice and overlap two workers. Check exactly one source-linked purchase, one installment, the correct statement and closing month, no account transaction, and unchanged account balance.
2. Create a confirmation-based rule. When due, check one expected upcoming item with no obligation or budget Expected spending. Confirm at R$ 165 on the 13th with a different eligible card/category for that occurrence. Check the rule and next month still show the original template, while the purchase uses the chosen values and statement.
   Reject a confirmation dated after today's São Paulo business date without changing the occurrence or any financial value.
   Force a retryable purchase failure after valid choices are submitted. Check no partial purchase, the failed item still shows R$ 165, the 13th, and the chosen card/category, then retry without those fields. Check the resulting purchase uses the retained choices and appears once. Explicitly changing a choice on retry requires fresh validation. Overlap a second confirmation with different amount/date/card/category while the first claim is active; check a retryable conflict and that the winning purchase and occurrence agree on every actual value. Interrupt a claimed action, check purchase existence, then safely retry without duplication.
3. Force automatic over-limit. Check no purchase and an awaiting-approval item with resulting negative credit explained. Approve explicitly after current-credit recheck; repeat the approval and check one purchase. Reject stale available-credit confirmation.
   In a separate occurrence, dismiss the awaiting-approval item; verify no purchase, no future regeneration of that date, and unchanged later cycles.
4. Pause/end a rule after an item becomes due; confirm or dismiss that item without reviving future dates. Let a day-12 charge become due while processing is down, then edit amount/card/schedule on day 13. Check day 12 is represented with the old values before the edit applies, and the next eligible date uses the new definition. Repeat with pause/end. If a due date cannot be represented, check the rule mutation returns a retryable conflict and leaves the old definition intact. Archive a selected card/category, repair future rule association, and explicitly override an already presented item while retaining its original identity.
   Process three missed dates with a recoverable purchase failure on the middle date. Check the first date stays committed, the middle date is visible as failed, and the third date is processed. If the middle date cannot even be recorded as failed, check processing stops that rule without advancing past it; earlier commits remain valid.
5. Backdate an automatic purchase into an already closed/paid statement. Check the original statement and recognition month restate once; effective payments are unchanged. Correct/refund the purchase and ensure the source date never regenerates.
6. Compare account recurrences, new card recurrences, manual card purchases, statement payments, transfers, Budget, Dashboard, and financial history for single recognition and unchanged existing semantics.

## Outcome measurement

- SC-001 and SC-006: Recruit at least 20 representative owners who use recurring expenses, including mobile and desktop users. Give each the same eligible card and gym scenario without coaching. Time from opening the recurrence form to saved-rule readback; at least 19 of 20 must finish within 3 minutes and correctly name the card, amount, mode, and next date. Separately show expected, recorded, externally unverified, and installment examples; time from display to correct classification, with at least 18 of 20 completing within 30 seconds. Record device, theme, elapsed time, and errors without collecting card credentials.
- SC-002–SC-005: Execute at least 100 fixture-driven cases or attempts for each criterion, including the named calendar, concurrency, statement, and lifecycle boundaries; record input, expected result, and actual result. Concurrent attempts must overlap in the test harness, not merely run sequentially.
- SC-007: Seed 1,000 owned rules with representative account/card mix and occurrences. In a browser at supported desktop and narrow mobile viewports, measure from list/detail navigation until primary content and controls are usable. Collect at least 20 timings per view and viewport after warmup under recorded representative device/network conditions; each p95 must be at most 2 seconds. Also check backend query counts to detect per-rule lookups.
