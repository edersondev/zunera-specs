# Backend Gates: Notifications

## Foundation — 2026-09-29

Status: **PASS for persistence and evaluator infrastructure only**. Source-specific projection and API behavior remain in T019–T049. No frontend work may start.

- Branches: `017-notifications` in specs, backend, and frontend; clean before edits (T001).
- Schema: three notification migrations applied successfully to upgraded local MySQL; the same migrations ran in fresh SQLite and isolated MySQL test databases.
- Tests: `php artisan test tests/Unit/Notifications tests/Feature/Notifications` passed on isolated MySQL: 13 tests, 40 assertions. The same files passed on SQLite in focused runs.
- Style: `vendor/bin/pint --test --format=agent` passed for all new backend notification files, migrations, and `routes/console.php`.
- Scheduler: `zunera-backend-scheduler-1` running `schedule:work`; log recorded `notifications:reconcile` at 18:17 UTC; `schedule:list` shows each minute; cache heartbeat recorded `2026-09-29T18:17:00+00:00`.
- Retry behavior: unknown source facts remain pending with bounded retry and no financial text in the error code. Story-specific projectors must replace the provisional dispatcher path before the feature-wide backend gate T049.

This gate does not assert matrix-event delivery, API contract, owner counts, source transitions, or five-minute timeliness. Those checks belong to the backend story and feature-wide gates.

## Backend US1 center API — 2026-09-29

Status: **PASS for seeded notification history and owner-scoped center contract**. Source-domain event projection remains in later backend stories.

- Contract and lifecycle tests on isolated MySQL, plus session lifetime regression: 10 tests, 58 assertions passed. SQLite focused tests passed too.
- Five protected routes registered: list, summary, read-all, individual read, and open.
- Coverage includes validation, locale fallback, stable cursor, 99/100 unread, 90-day retention, foreign item 404, immediate source-ownership loss from list/count/open, unseen-page read-all, and read without paying a statement.
- Pint check passed for center service, controller, request/resource, tests, and routes.
- Financial integrity: API read/open/read-all only update notification read timestamps; the statement payment amount remains unchanged in the contract test.

No frontend work may start before T049 passes.

## Backend US2 statement deadlines — 2026-09-29

Status: **PASS** for source-authoritative statement stages and payment resolution.

- Isolated MySQL statement, scheduler, and center authorization tests: 16 passed, 76 assertions. Credit Cards regression after payment, credit, purchase, and correction hooks: 289 passed, 2,427 assertions.
- Repeated evaluation of the same statement stage creates one event. Tests cover three approaching entry days, due today, overdue, payment/credit resolution, partial payment, reversal, open/zero exclusion, business midnight, and presentation timezone changes.
- Accepted statement mutations capture durable facts in the same transaction. `CreditCardStatementService::syncStatement` also captures a fact; its read-only list/detail/refresh paths remain presentation reconciliation and do not create notification facts.
- Scheduler boundary test advances an unpaid statement across a Sao Paulo business day and runs the minute command offline: stage appears in the first pass (test command time about 0.03 seconds). The live scheduler tick interval observed in the foundation gate is one minute; production end-to-end lag remains subject to T048 measurement.
- Pint passed after import ordering was fixed. No frontend work may start before T049 passes.

## Backend US3 recurrence review — 2026-09-29

Status: **PASS** for exact occurrence review and source-side resolution.

- Isolated MySQL recurrence, transaction, and notification regression: 471 passed, 3,939 assertions. Pint passed for all changed recurrence source/projector files and the new notification test.
- Ordinary generated pending transaction and confirmation-mode card Expected each capture a fact in the accepted transaction. Automatic card purchase success creates no review item; over-limit and Failed states do.
- Expected→Failed retains one `recurrence_review` event keyed to the occurrence ID and updates the current summary to retry context. Internal failure codes and issuer-charge claims are absent. Dismiss, recorded purchase, effective/removed generated transaction resolve the same item; pending restore reuses its identity.
- Rule pause leaves earlier occurrence review active. The API destination names the exact generated transaction or card occurrence.

## Backend US4 budget thresholds — 2026-09-29

Status: **PASS** for current-month category-plan stages against the source calculation.

- Default SQLite budget, Credit Cards, transaction, and notification regressions: 387 passed, 3,277 assertions. Focused isolated MySQL budget and scheduler notification tests: 9 passed, 37 assertions. Pint passed for changed files.
- `BudgetCalculationService::forMonth` provides realized status; projected spend and overall monthly utilization are excluded. Tests verify 80%, reached, exceeded, direct jump with lower-stage consumption, fall/recross identity, and no new stage after month end.
- Accepted plan, transaction, card purchase, correction, and credit-event mutations capture affected month facts. An accepted expense created an approaching notification with the exact realized source amount in the owner center. Minute scheduler test resolves the prior month and scans the new current month after Sao Paulo midnight.
- A broad isolated MySQL run exposed two existing SQLite-oriented test assumptions: `BudgetPerformanceTest` hardcodes monthly budget ID 1 and SQLite `EXPLAIN QUERY PLAN`; `CreditEventRecognitionTest` passes a collection key as `intval`'s base when MySQL returns numeric strings. The latter mismatch reproduces with the pre-notification credit-event service, while the same affected suites pass on their default SQLite setup. Feature-wide MySQL checks remain in T046–T049.

## Backend US5 goal milestone — 2026-09-29

Status: **PASS** for one informational event per target state.

- Focused isolated MySQL acceptance: 4 passed, 41 assertions. Default goal and notification suite: 65 passed, 1 skipped, 522 assertions. Pint passed.
- Accepted create, allocation, withdrawal, target edit, and lifecycle operations capture facts inside the idempotency transaction. A brief reach survives a withdrawal before fact processing.
- Falling and regaining the same target retains one event. Reaching a changed target creates a second identity. Archived/completed goals do not qualify anew. The milestone leaves goal status active, account balance unchanged, and requires-action count zero.

## Backend US6 category preferences — 2026-09-29

Status: **PASS** for owner-scoped preference API and first-qualification timing.

- Isolated MySQL notification suite: 50 passed, 318 assertions, including the real concurrent preference/fact test across statement, recurrence, budget, and goal sources. Pint passed for the preference and precision files.
- GET returns four enabled defaults. PATCH accepts only a known category and a boolean `enabled`; invalid category, extra fields, and non-boolean input fail. Other owners' settings and event history remain inaccessible.
- A qualifying fact and preference update serialize on the owner row. Microsecond `effective_at` and `qualified_at` preserve their order, including overlapping requests. A disabled first qualification records suppressed identity; enabling later does not backfill it. Critical events obey their category switch.

## Feature-wide backend gate — 2026-09-29

Status: **PASS** for backend contracts, domain integration, concurrency, authorization, retention, exclusions, and style. Frontend implementation may begin.

- Full default SQLite backend suite: **1,347 tests, 55,763 assertions, 6 skips, 0 failures** using `php -d memory_limit=512M vendor/bin/phpunit --no-progress`. The ordinary `php artisan test` process uses a 128 MB limit and exhausts it in the existing 5,000-transaction scale test; the direct PHPUnit invocation applies the larger CLI limit to that process.
- Isolated real MySQL notification feature and unit suite: **61 passed, 2,474 assertions**, including four-worker simultaneous evaluation, overlapping preference/fact order for all four sources, 10,000-item owner/cursor/count scale, source authorization, retention/redaction, injected failure recovery, and excluded financial operations.
- The 10,000-item MySQL test served 20 distinct equal-timestamp cursor pages of 50 items, preserved owner counts and filters, and met the 19/20 under-three-second page target. Twenty live/simulated delivery cases met the five-minute target; exact start/response times and the simulated-midnight limitation are in `timeliness-performance.md`.
- `vendor/bin/pint --test --format=agent` passed across the backend. One pre-existing date-boundary card test had a random credit limit that could be below its fixed purchase amount; its fixture now specifies a sufficient limit, and the full suite passes.
- The broad MySQL run from the US4 gate still has two existing SQLite-specific test assumptions in `BudgetPerformanceTest` and `CreditEventRecognitionTest`. The notification MySQL suite passes; the full backend suite passes on its configured default SQLite connection.

## Review remediation — 2026-09-30

- Statement date-boundary instants are converted from Sao Paulo business time to UTC before persistence. `StatementNotificationTest::date_scan_records_business_midnight_as_a_utc_instant` checks both the stored value and serialized instant.
- The minute statement scan processes current due-window candidates before a bounded older-overdue backlog and excludes paid statements. `StatementNotificationTest::date_scan_prioritizes_new_due_statements_over_old_overdue_backlog` checks that a new due item appears in the first limited pass and the old backlog still advances.
- The updated statement test file passed: 10 tests, 35 assertions. The scheduler test file passed: 5 tests, 21 assertions. The complete default SQLite backend suite passed: **1,349 tests, 55,770 assertions, 6 skips** using the documented 512 MB PHP CLI limit. Pint passed on all four modified PHP files.
- These regression tests prove ordering and UTC storage. A new live high-volume MySQL statement-scan latency sample was not collected; the prior 10,000-event MySQL benchmark measured notification list performance, not this scan.
