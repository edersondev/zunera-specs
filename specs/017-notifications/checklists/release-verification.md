# Notifications Release Verification

Last checked: 2026-09-30. Branch: `017-notifications` in specs, backend, and frontend.

## Automated gates

| Gate | Outcome | Evidence |
|---|---|---|
| Backend complete default SQLite suite | Pass | After review fixes, `php -d memory_limit=512M vendor/bin/phpunit --no-progress`: 1,349 tests, 55,770 assertions, six skips. The direct invocation supplies sufficient memory for the existing 5,000-transaction test. |
| Backend real MySQL scale | Pass | Isolated 10,000-owner-event test: 20 first-page/filter requests, 20/20 below three seconds, 19.9 ms 95th percentile; 20 distinct cursor pages and owner count verified. |
| Backend style | Pass | `vendor/bin/pint --test --format=agent`: passed. |
| Frontend complete unit suite | Pass | After review fixes, `npm run test:unit -- --run`: 159 files, 588 tests passed. |
| Frontend lint and build | Pass | `npm run lint`: clean. `npm run build`: passed; Vite reported existing large-chunk advisories. |
| Isolated notification and affected browser regressions | Pass | Earlier Chromium run: 57 tests across notification, performance, credit card, budget, recurrence, and financial goal specs. After review fixes, the isolated notification file passed 36/36 across Chromium, Firefox, and WebKit. |
| Browser 10,000-item fixture | Pass with scope limit | 20 center openings/filter changes, 20/20 usable under three seconds; release-run 95th percentile 272.4 ms. Browser API was mocked; see [timeliness-performance.md](timeliness-performance.md). |
| Scheduler runtime | Pass | Docker app, MySQL, and `zunera-backend-scheduler-1` running; cache heartbeat advanced to `2026-09-30T14:04:00+00:00` across the date rollover. |

The backend MySQL notification suite, source mutation, authorization, privacy, and concurrency evidence from the earlier gate is in [backend-gates.md](backend-gates.md). The 20 delivery samples, simulated-midnight limitation, and failure/recovery check are in [timeliness-performance.md](timeliness-performance.md).

The review fixes add UTC boundary and scan-priority backend tests plus frontend tests for stale read-all responses, unknown preference state, unavailable-source labels, and date grouping. The new scan path has not had a live high-volume MySQL timing run; the existing MySQL scale result covers list/filter responses.

## Outcome status

- SC-002, SC-003, and SC-004: automated backend and frontend acceptance checks pass as recorded in the backend gate and browser suite.
- SC-005: 20/20 measured delivery cases met five minutes. Two business-midnight cases used an injected clock; unattended real-midnight timing is still unobserved.
- SC-006: real MySQL and browser 10,000-item components each pass separately. A direct browser run against the seeded MySQL dataset remains unmeasured; do not treat the combined result as a measured integrated path.
- SC-001 and SC-007: pending human-participant checks in [usability.md](usability.md).

The feature code is implemented and automated gates pass. Release acceptance remains open for the human-participant outcomes, unattended real-midnight timing, and an integrated browser measurement against the seeded MySQL dataset. The separate scale and simulated-boundary results above support the implementation but do not replace those direct observations.
