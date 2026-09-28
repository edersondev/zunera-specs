# Financial Reports performance evidence

**SC-005 status: met on the measured live fixture.** All 260/260 complete browser-to-API-to-MySQL attempts reached the requested source rows within five seconds. Every sampled class passed 20/20, above the 95% target.

## Complete live journeys — 2026-09-27

Chromium ran against the local Vue development server and the running Laravel API/MySQL stack. A disposable authenticated account owned 10,000 fictional effective transactions across 100 categories and 50 accounts, 100 effective transfers, and one effective card statement settlement. The current and prior periods both contained records; one expense category contained one contributor. Each attempt began before browser navigation to Reports and ended after the requested contribution rows appeared in the drawer. Each API detail revision was checked against its overview revision; the deeper-page attempt also checked the later page revision and visible appended rows. No API response was mocked. The disposable account and all its records were removed after measurement.

| Requested displayed-total class/path | Attempts ≤ 5 s | Median | 95th sample | Maximum |
|---|---:|---:|---:|---:|
| Summary income | 20/20 | 1.304 s | 1.497 s | 1.664 s |
| Summary expenses | 20/20 | 1.326 s | 1.482 s | 1.520 s |
| Summary result | 20/20 | 1.343 s | 1.497 s | 1.503 s |
| Expense category | 20/20 | 1.296 s | 1.458 s | 1.486 s |
| Income category | 20/20 | 1.371 s | 1.493 s | 1.703 s |
| One-contributor expense category | 20/20 | 1.468 s | 1.523 s | 1.535 s |
| Account direct expense | 20/20 | 1.506 s | 1.580 s | 1.599 s |
| Account transfer in | 20/20 | 1.430 s | 1.494 s | 1.498 s |
| Account transfer out | 20/20 | 1.448 s | 1.484 s | 1.521 s |
| Account card settlement | 20/20 | 1.371 s | 1.516 s | 1.722 s |
| Current comparison expense | 20/20 | 1.471 s | 1.522 s | 1.532 s |
| Prior comparison expense | 20/20 | 1.418 s | 1.478 s | 1.494 s |
| Summary expense, second cursor page | 20/20 | 1.787 s | 1.852 s | 1.873 s |
| **All classes** | **260/260 (100%)** | — | — | **1.873 s** |

The first 240 attempts (all classes except settlement) had an overall median of 1.431 s and 95th sample of 1.784 s. The settlement's 20 attempts were measured separately after extending the same fixture. These are local development-stack measurements, not a production latency claim. Browser automation used Playwright with a temporary measurement script; the permanent backend fixture and command below provide revision-cost and scale checks.

## Revision measurements — 2026-09-27

The isolated MySQL test fixture contains 10,000 effective transactions, 100 categories, and 50 accounts. After one warm request, 20 consecutive backend request samples for each path and 20 direct source-revision calculations gave:

| Backend operation | Median | 95th sample | Maximum | Samples under 5 s |
|---|---:|---:|---:|---:|
| Complete overview, including source revision | 0.416 s | 0.427 s | 0.429 s | 20/20 |
| First 100-row expense contribution page, including source revision | 0.196 s | 0.201 s | 0.214 s | 20/20 |
| Source-revision calculation alone | 0.122 s | 0.129 s | 0.142 s | 20/20 |

The fixture test passed with 56 assertions and confirmed the second cursor page and `transactions_owner_history_index` use. Command: `docker exec zunera-backend-app-1 sh -c 'set -a; . ./.env.testing; set +a; REPORT_PERF_LOG=1 php artisan test --filter=FinancialReportPerformanceTest'`. These are in-process backend timings with a live MySQL source; the complete journey evidence is recorded above.

## Baseline measurements — 2026-09-26

Measured on 2026-09-26 (America/Sao_Paulo). The backend fixture creates 10,000 effective transactions across 100 expense categories and 50 accounts in the isolated MySQL test database. The frontend browser fixture supplies the corresponding 100 category and 50 account aggregates and 100-row cursor pages representing 10,000 contributions.

| Check | Result | Target |
|---|---:|---:|
| MySQL warm overview request | 0.292 s | < 5 s |
| MySQL first 100-row contribution page | 0.074 s | < 5 s |
| Overview expense total | 1,000,000 centavos across 100 categories | Exact |
| Account rows | 50 | 50 |
| Source query plan | `transactions_owner_history_index` selected | Indexed |
| Browser warm navigation plus first detail page with mocked API, 20 attempts each in Chromium, Firefox, WebKit | 60/60 under 5 s (100%) | Frontend-only check |
| Browser timing overall median / 95th percentile / maximum | 1.148 s / 1.449 s / 1.597 s | < 5 s |

Backend command: `docker exec zunera-backend-app-1 sh -c 'set -a; . ./.env.testing; set +a; REPORT_PERF_LOG=1 php artisan test --filter=FinancialReportPerformanceTest'`. Passed with 16 assertions. This also checked the all-record detail total, first and second cursor pages, and index use.

Browser command: `CI=1 npm run test:e2e -- e2e/financial-reports.spec.js`. The browser timings include selecting a period, rendering summary/category/account sections, opening the first source page, and closing it. Browser API responses were mocked at the verified contract boundary, so these timings measure frontend rendering and navigation separately from the MySQL request timings; they are not a single end-to-end latency sample.

Latest observed ranges: Chromium 737–1088 ms (95th percentile 1079 ms); Firefox 725–1433 ms (95th percentile 1229 ms); WebKit 1085–1597 ms (95th percentile 1489 ms). Each browser passed all 20 attempts.
