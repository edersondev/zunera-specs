# Notification Timeliness and Performance Evidence

## Backend delivery — 2026-09-29

Environment: local Docker `zunera-backend-app-1`, MySQL 8.4, and running `zunera-backend-scheduler-1` (`schedule:work`, one-minute `notifications:reconcile`). A dedicated owner logged in through the live HTTP app with session and CSRF cookies. Each visibility time is the first authorized `GET /api/v1/notifications` response containing the exact source destination. The benchmark owner and its financial fixtures were deleted afterward. The reproducible harnesses are `tests/Support/notification_timeliness_benchmark.php` and `tests/Support/notification_boundary_benchmark.php` in the backend.

For accepted goal creation, the start is immediately before the POST, so measured latency includes the source request and conservatively precedes its commit. For recurrence, it is immediately before `recurring:process-due --date=2026-09-29`. For the last two rows, the Sao Paulo 2026-09-30 midnight boundary is simulated in the evaluator process; the start is when that simulation begins, and visibility is observed through the separate live HTTP process. They verify boundary logic and HTTP visibility, while the live scheduler proxy observations below verify minute-tick pickup. This does not measure an unattended real midnight transition.

| Case | Source / condition | Start UTC | First authorized response UTC | Seconds |
|---|---|---|---|---:|
| 1 | Goal create 3 | 19:32:46.822577 | 19:32:46.902399 | 0.080 |
| 2 | Goal create 4 | 19:32:46.902436 | 19:32:46.952010 | 0.050 |
| 3 | Goal create 5 | 19:32:46.952034 | 19:32:46.998326 | 0.046 |
| 4 | Goal create 6 | 19:32:46.998347 | 19:32:47.048612 | 0.050 |
| 5 | Goal create 7 | 19:32:47.048634 | 19:32:47.096436 | 0.048 |
| 6 | Goal create 8 | 19:32:47.096458 | 19:32:47.145382 | 0.049 |
| 7 | Goal create 9 | 19:32:47.145404 | 19:32:47.197257 | 0.052 |
| 8 | Goal create 10 | 19:32:47.197280 | 19:32:47.246970 | 0.050 |
| 9 | Goal create 11 | 19:32:47.246992 | 19:32:47.298045 | 0.051 |
| 10 | Goal create 12 | 19:32:47.298077 | 19:32:47.349386 | 0.051 |
| 11 | Goal create 13 | 19:32:47.349409 | 19:32:47.400924 | 0.052 |
| 12 | Goal create 14 | 19:32:47.400962 | 19:32:47.452815 | 0.052 |
| 13 | Goal create 15 | 19:32:47.452838 | 19:32:47.505529 | 0.053 |
| 14 | Goal create 16 | 19:32:47.505559 | 19:32:47.556953 | 0.051 |
| 15 | Goal create 17 | 19:32:47.556984 | 19:32:47.613418 | 0.056 |
| 16 | Goal create 18 | 19:32:47.613440 | 19:32:47.668071 | 0.055 |
| 17 | Goal create 19 | 19:32:47.668100 | 19:32:47.720851 | 0.053 |
| 18 | Recurrence processor, transaction 20007 | 19:32:47.736189 | 19:32:47.845094 | 0.109 |
| 19 | Simulated midnight, due today, statement 7 | 19:35:42.891382 | 19:35:42.937213 | 0.046 |
| 20 | Simulated midnight, overdue, statement 8 | 19:35:42.891382 | 19:35:42.937210 | 0.046 |

All timestamps in this table are on **2026-09-29 UTC**. Result: **20/20 within five minutes** (100%; target at least 95%). The date-stage rows use an injected clock as described above.

Live scheduler pickup was separately observed without an injected clock: a due-today statement seeded at 19:32:47.853895 UTC first appeared at 19:33:00.533190 UTC (12.679 s); an overdue statement seeded at 19:33:00.536275 UTC first appeared at 19:34:00.660027 UTC (60.124 s). These start times are fixture availability, not actual midnight boundaries.

The 2026-09-30 review fix prioritizes current due-window statements ahead of older overdue backlog and normalizes date-boundary instants to UTC. The measurements above predate that fix. A focused regression test confirms first-pass priority with a two-row limit; a live high-volume MySQL scan latency measurement remains open.

Injected failure/recovery: `NotificationSchedulerTest::injected_evaluator_failure_recovers_without_duplicate_delivery` creates a qualified goal fact, deliberately throws from the evaluator, confirms one retry attempt and zero events, makes the fact available, then drains twice. It finishes with one processed fact and exactly one `goal_reached` event. The focused SQLite test passed (7 assertions); the MySQL run is included in the final backend gate.

## 10,000-item first-page and browser benchmark — 2026-09-29

The real MySQL backend scale test seeded 10,000 retained owner events plus one foreign event and made 20 authenticated first-page requests, alternating All and Unread filters with a 25-item limit. It also verified 20 distinct 50-item cursor pages and owner counts. Run: `NOTIFICATION_SCALE_REPORT=1 php -d memory_limit=512M vendor/bin/phpunit --no-configuration --bootstrap vendor/autoload.php --no-progress --filter=ten_thousand_mysql_items_keep_owner_counts_and_equal_time_cursor_fast tests/Feature/Notifications/NotificationConcurrencyAndScaleTest.php` inside the backend app container. All 20 first-page/filter responses were below three seconds; 95th percentile **19.9 ms**, maximum **19.9 ms**. Sorted response samples in milliseconds: 15.7, 15.7, 15.9, 16.0, 16.3, 16.4, 16.6, 17.2, 17.2, 17.5, 17.5, 17.9, 17.9, 18.0, 18.3, 18.4, 18.9, 19.5, 19.9, 19.9. These are Laravel in-process request times, including authorization and serialization, without browser/network transport.

The isolated Chromium benchmark `e2e/notifications-performance.spec.js` held 10,000 retained items in its API fixture and returned the same 25-item page shape. It repeated 20 full center openings, alternating All and Unread, and waited until the newest item was visible. API resource response 95th percentile was **17.3 ms** (maximum 20.4 ms). Browser usable-content 95th percentile was **276.8 ms** (maximum 336.7 ms). All 20 attempts showed content in less than three seconds. Browser usable-content samples in milliseconds, in attempt order: 336.7, 244.9, 188.6, 276.8, 185.0, 256.1, 209.5, 265.7, 174.7, 253.8, 185.5, 260.9, 181.4, 274.4, 189.3, 239.2, 185.3, 242.0, 192.5, 254.5.

The backend and browser measurements ran separately: the browser used a mocked API, so these samples do not constitute a single end-to-end browser request against the seeded MySQL database. They separately exercise the real 10,000-item database query and the browser's first-page rendering at that scale. An integrated live-browser/seeded-MySQL 20-run sample remains to be measured for a direct SC-006 claim.
