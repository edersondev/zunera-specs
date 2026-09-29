# Financial Reports revision verification

## Backend gate T013 — 2026-09-27

Branch: `017-financial-reports` in specs, backend, and frontend. Backend response contract version: `1.1.0` in `contracts/financial-reports-api.yaml`. Both protected responses now include an opaque, scope-bound `source_revision`; each response reads money, source rows, labels, and revision within one database transaction. The existing validation, authorization, resource, and recognition paths remain in use. Frontend revision work had not begun when this gate passed.

| Check | Result |
|---|---|
| `php artisan test --compact --filter=FinancialReport` in Docker | 33 passed, 2 MySQL-only tests skipped, 306 assertions |
| MySQL `.env.testing` `php artisan test --compact --filter=FinancialReportConcurrentSnapshotTest` | 2 passed, 18 assertions; committed concurrent writes between internal reads |
| `php artisan test --compact --filter='CreditCard\|FinancialDashboard\|Budget'` in Docker | 498 passed, 50,060 assertions |
| `php artisan test --compact` in Docker | 1,283 passed, 3 skipped, 55,396 assertions |
| `vendor/bin/pint --format=agent` in Docker | Passed; formatted the new MySQL snapshot test |

The Docker container does not expose Git metadata to Pint, so `--dirty` was unavailable. The full Pint command completed. The three full-suite skips include the two SQLite runs of MySQL-only snapshot tests. Focused MySQL execution passed separately.

T008–T012 assertion mapping and confirmed category-source rule are recorded in [revision-audit.md](revision-audit.md). Backend gate passed before frontend revision edits.

## Post-implementation checks — 2026-09-27

| Check | Result |
|---|---|
| Full Laravel suite after revision and performance-test changes | 1,283 passed, 3 skipped, 55,396 assertions |
| MySQL concurrent-snapshot regressions | 2 passed, 18 assertions |
| Pint after final backend edits | Passed |
| Full frontend Vitest suite after final strict-scope adjustment | 148 files, 560 tests passed |
| Frontend build after final strict-scope adjustment | Passed; existing bundle-size warning |
| Frontend standard lint after final strict-scope adjustment | Passed |
| Full isolated Playwright financial-reports suite before final strict-scope edit | 51 passed across Chromium, Firefox, and WebKit |
| Focused Playwright source-change journeys after final build | 6 passed across the three browsers |
| Complete live performance study | SC-005 met: 260/260 browser/API/MySQL journeys within five seconds; see [performance.md](performance.md) |

The delivered overview and detail responses match contract version 1.1.0: protected scope metadata, opaque `source_revision`, exact-centavo totals, section states, signed contribution rows, and cursor paging. The browser requires matching revisions and applied scopes, restarts at the first page after a stale later page, and offers a retry after three unstable paired reads. Spec, plan, data model, contract, quickstart, and task state describe this behavior. The final T028 gate remains open until the 10-person uncoached study establishes SC-002 and SC-003; these checks must be repeated after any study-driven product changes.
