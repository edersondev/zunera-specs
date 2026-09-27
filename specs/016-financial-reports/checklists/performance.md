# Financial Reports performance evidence

**SC-005 status: pending.** These separate measurements do not establish the full summary-to-contributions journey with a live API. Run at least 20 representative integrated attempts against the 10,000-record fixture and confirm at least 95% finish within 5 seconds before marking SC-005 met.

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
