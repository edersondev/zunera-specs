# Financial Reports implementation reconciliation

Verified on branch `016-financial-reports` in specs, backend, and frontend. No report ledger, balance table, new package, or public route was added.

| Requirement area | Implementation and evidence |
|---|---|
| Recognized money and parity (FR-002, FR-005–FR-012, FR-029, SC-001) | Shared card credit-event allocation feeds Reports, Dashboard, Budgets, and Financial History. Feature and unit fixtures cover effective ordinary activity, pending/removed activity, transfers, goals, recurring generation, statement closing/payment, full/partial paid refunds, and centavo reconciliation. Full Laravel suite passed: 1,274 tests, 55,327 assertions, one existing skip. |
| Periods and comparison (FR-003–FR-004, FR-020–FR-021, SC-004, SC-006) | One Sao Paulo resolver returns current/prior inclusive dates and durations; URL scope restores presets, historical month, custom dates, and filters. Unit and browser tests cover quick choices, historical versus full-month custom comparison, partial dates, leap/short months, negative and zero denominators. |
| Categories, accounts, filters, and source detail (FR-013–FR-018, FR-022–FR-027) | Backend overview and cursor detail use one owner-scoped contribution read model; account movements remain separate. Frontend has numeric chart alternative, ranked categories, archived labels, filter chips, signed paged drawer, and source links to transaction, statement, or transfer detail. Browser tests cover paid refund trace, filter scope, empty/unavailable states, and foreign IDs. |
| Contract and security (FR-001, SQR-001–SQR-003) | Both protected response shapes validated against `contracts/financial-reports-api.yaml`; backend authentication, owner ID, invalid range, metric, and cursor tests pass. Isolated MySQL tests verify paid refunds and paging. |
| Localization, accessibility, and scale (FR-028, SC-005) | PT-BR/English copy, numeric tables, keyboard drawer close, themes, 320 CSS pixel layout, and 200% zoom equivalent layout are covered in browser tests. Final frontend unit suite passed 547 tests. Lint and production build pass. Reports Playwright passed 39/39 across Chromium, Firefox, and WebKit before the last focused edits; affected journeys passed again afterward. Scale measurements are in [performance.md](performance.md). |

Human outcomes SC-002 and SC-003 remain pending because no participant results exist. The study template is in [usability.md](usability.md); T066 remains unchecked.
