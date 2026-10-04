# Implementation Plan: Restore Account Data

**Branch**: `021-restore-account-data` | **Spec**: [spec.md](spec.md)

## Backend first

1. Add protected `POST /api/v1/account-data/archives/{archive_id}/restore` with an empty request. Return `200` with `restored_archive_id`, `restored_record_count`, and nullable `previous_archive_id`; use owner-scoped `404` and safe-restore `409` errors.
2. Reuse account-data snapshot and clear operations inside one transaction. Lock the user, verify selected archive, snapshot current financial data only when present, clear live data, replay archive, and keep selected archive unchanged.
3. Replay records in dependency order with fresh IDs, remapped relationships, and a deferred recurring-purchase link. Map saved system categories to live defaults, omit generated columns, and reject invalid snapshots. Rebuild notification projection facts after replay.
4. Test linked round trip, access, repeated and empty restore, rollback, and regressions; run Pint and full backend suite before frontend.

## Frontend after backend contract

1. Add Restore to each archive row and a focused confirmation dialog. Explain current-data backup and selected-archive retention, with localized error feedback.
2. Use the existing Axios account-data service and reload dashboard after success to clear cached workspace state. Keep archive detail read-only.
3. Follow design foundation, app shell, navigation, and component guidance. Run unit tests, focused Playwright at 320 px, build, ESLint, and Oxlint.

## Verification

- Backend: 1,376 PHPUnit tests passed, 6 skipped; focused restore and reset tests and Pint passed.
- Frontend: 617 Vitest tests passed; focused Playwright (3 browser projects), build, ESLint, and Oxlint passed.
- No migration or new package required.
