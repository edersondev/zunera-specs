# Implementation Plan: Reset Account Data

**Branch**: `020-reset-account-data` | **Spec**: [spec.md](spec.md)

## Backend first

1. Add `financial_data_archives` and `financial_data_archive_records` with owner-scoped, immutable JSON snapshots and indexed record type/source IDs. The migration is additive and reversible.
2. Add protected endpoints: `POST /api/v1/account-data/archive` returns `201` archive metadata; `DELETE /api/v1/account-data` accepts `{current_password}` and returns `204`; `GET /api/v1/account-data/archives` lists owner archives; `GET /api/v1/account-data/archives/{archive_id}/records?type=&page=` returns 25 read-only records per page.
3. Use Form Requests for mutation and listing input, plus one account-data service. Lock the user, snapshot owned rows, clear derived records, and delete live financial rows in foreign-key order inside a transaction. Delete also removes prior archives. Reuse current-password error and rate-limit behavior.
4. Test authorization, relationship fidelity, delete order, password failure and limits with the database-backed cache, owner isolation, and preserved login. Run Pint and backend suite.

## Frontend after backend contract

1. Add Account data to the header menu and route to a settings page with separate archive and delete cards.
2. Use one focused confirmation dialog. Delete alone requires current password and explains permanence; archive explains read-only access. Keep the dialog accessible and clear sensitive input on close.
3. Add archive list and paginated read-only records, with localized labels and BRL amounts. Route all requests through an Axios service; reload dashboard after reset to clear cached Pinia data.
4. Run Vitest, focused Chromium Playwright journey, build, ESLint, and Oxlint. Follow design foundation, app shell, navigation, and component guidance.

## Verification and rollout

- Backend suite: 1,369 tests passed, 6 skipped, using 512 MB CLI memory for the existing 10,000-record performance test. A regression test confirms failed password attempts persist with the database-backed limiter cache.
- Frontend suite: 614 tests passed. Focused Chromium browser flow, build, ESLint, Oxlint, and Pint passed.
- Applied the new migration to the live development container. Deploy the additive migration before serving the new backend endpoint.
