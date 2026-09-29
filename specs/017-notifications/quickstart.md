# Quickstart: Notifications

## Before implementation

1. Verify `017-notifications` is active in specs, `../zunera-backend`, and `../zunera-frontend`. Confirm working-tree changes before editing.
2. Read [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [plan.md](plan.md), and [the API contract](contracts/notifications-api.yaml).
3. Read `docs/design/design-foundation.md`, `docs/design/app-shell.md`, `docs/design/navigation.md`, and `docs/design/components.md`. Complete backend contract, authorization, validation, and tests first.

## Backend sequence

1. Add durable event identity, preference/current change history, and projection-request storage. Unique user/event identity must survive suppression and visible-content expiry. Add owner-scoped indexes for center, count, and action views.
2. Add source adapters that read Card reconciler/state, Budget calculation, goal projection, ordinary generated transaction, and card occurrence. They must never recompute financial values. Add protected routes, Form Requests, Resources, thin controllers, and safe typed destination resolution matching the contract.
3. Write a projection fact inside each accepted source mutation transaction after authoritative state is settled; consume it after commit. Capture any newly qualified type/stage and event time so delayed processing can use the preference then. Cover all mutations listed in the plan. Reconcile old actionable events as source conditions change.
4. Add minute scheduler, bounded request drain, date-stage evaluation, recovery scan, retention cleanup, and non-sensitive lag/failure metrics. Add a running scheduler process to Docker/deployment; verify its heartbeat and a real due-boundary evaluation. Existing recurrence due processor remains authoritative.
5. Test each matrix row, source transitions, late eligibility, threshold jumps, recross/read behavior, goal target reuse, disabled/re-enabled preference timing, 90-day expiry, deletion/authorization loss, and navigation. Verify no notification action changes source financial state. Use MySQL for duplicate/concurrent projection and preference races; SQLite alone is insufficient.
6. Complete all six backend story increments and feature-wide concurrency, privacy, timeliness, routine-action exclusion, and full-suite checks. Gate every frontend task on the passing feature-wide backend gate T049 in [tasks.md](tasks.md).

Preferred backend checks from `../zunera-backend/` with development stack running:

```bash
docker exec zunera-backend-app-1 php artisan test --filter=Notifications
docker exec zunera-backend-app-1 php artisan test --filter=CreditCard
docker exec zunera-backend-app-1 php artisan test --filter=Budget
docker exec zunera-backend-app-1 php artisan test --filter=Recurring
docker exec zunera-backend-app-1 php artisan test --filter=FinancialGoal
docker exec zunera-backend-app-1 php artisan test
docker exec zunera-backend-app-1 vendor/bin/pint --format=agent
```

Docker socket may require sandbox escalation. Verify schema migration on fresh SQLite and upgraded MySQL databases. Run the new notification reconciliation command in an isolated test environment; do not create real user alerts in the shared development database just to smoke-test.

## Frontend sequence

1. Implement `notificationService.js` and service tests against the final backend contract. Add Pinia shared summary/center/preference state; clear it on logout and avoid stale responses after a session change.
2. Add protected center route, header indicator, simple views, paged list, and preference toggles. Use typed destination map and source availability rather than raw URLs. Add exact card-occurrence and budget month/plan route entry where missing.
3. Verify server-provided plain-text title/summary follow the shared `Accept-Language` header for PT-BR/English and fall back to PT-BR when missing or unsupported; localize control/count/read/action labels and format dates/amounts with existing rules. Use Element Plus standard controls and semantic theme tokens. Test keyboard/focus, screen-reader labels, responsive/high zoom, and light/dark/system themes.
4. Add service/store/component tests and isolated Playwright journey. Verify mark-read, mark-all, open, source resolution elsewhere, preference suppression, unavailable destination, and unread count. Verify read does not confirm/pay anything.

Frontend checks from `../zunera-frontend/`:

```bash
npm run test:unit -- --run
npm run lint
npm run build
CI=1 npm run test:e2e -- e2e/notifications.spec.js
```

## Acceptance walkthrough

1. New account: all four preferences on; no history shows clean empty state; header shows no count.
2. Create an eligible unpaid card statement. At 1–3 local days before due, see one approaching item; on due date, one due-today item; on overdue state, one critical item. Partial pay updates outstanding without another equivalent item; full pay resolves all. Reverse effective payment and verify retained current stage reactivates as read.
3. Generate ordinary pending occurrence and card expected occurrence. Each has one review item. Move card occurrence to a reviewable failed state and verify its existing item updates to retry context without duplicating. Open each exact review context; read alone changes no financial state. Confirm or dismiss from source feature; item resolves.
4. Move one current-month category plan from below 80% directly above 100%. Only exceeded alert appears. Drop and recross in same month; no new equivalent item. At month rollover, prior-month attention resolves; a later correction to that ended month generates no alert. New month has independent key. A projected-only overage creates no item.
5. Reach an active goal target, fall below, and regain the same target. One informational success item remains; goal is not automatically completed. Change target to a new amount and reach it for a distinct milestone.
6. Disable a category, first qualify an event, then re-enable. No backfill. Cause a new distinct stage/event and verify it may notify. Other category history stays visible.
7. Verify foreign/unauthorized IDs and removed source never leak details or navigate to a broken page. Mark all read across multiple pages; pending source actions remain pending. After 90 days, nonactionable visible content expires but event key is never reissued.

## Outcome measurement

- Timeliness: with application, database, and evaluator healthy, run at least 20 qualifying source-change and business-midnight cases, including late statement eligibility. Record accepted commit or boundary time and first authorized in-app response per case; at least 95% must be visible within five minutes. Test deliberately injected failure/recovery separately for eventual delivery without duplicates. Record scheduler heartbeat and projection lag without financial content.
- Volume: seed 10,000 retained items for one owner and test at least 20 first-page and filter-change attempts; at least 95% show usable recent items in three seconds. Verify cursor stability and exact count.
- Usability: with at least 10 representative participants, record uncoached time to find newest actionable item/destination and whether they distinguish read from resolved. At least 90% pass each applicable criterion; record anonymized results.
