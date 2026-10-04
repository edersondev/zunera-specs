# Feature Specification: Reset Account Data

**Feature Branch**: `020-reset-account-data`
**Backend Branch**: `020-reset-account-data` (`../zunera-backend`)
**Frontend Branch**: `020-reset-account-data` (`../zunera-frontend`)
**Status**: Implemented
**Input**: Let a signed-in user start fresh by archiving or permanently deleting all financial data.

## User Scenarios

### Archive all

A user confirms **Archive all** without entering a password. Their active financial workspace becomes empty. They can browse each saved archive and its records, but cannot restore or edit them from the archive view.

### Delete all

A user confirms **Delete all** only after entering their current password. The dialog explains that current financial data and earlier archives cannot be restored in Zunera. A successful request leaves their login and profile active and returns them to an empty workspace.

### Failure and access

- Wrong password or a rate limit leaves live data and archives unchanged.
- Unauthenticated users cannot access any account-data endpoint; users cannot read another user's archive.
- Archive or delete fails atomically if the database operation fails.
- Portuguese and English copy remains clear on narrow screens and with keyboard navigation.

## Requirements

- **FR-001**: Account menu opens an account-data page with distinct Archive all and Delete all actions.
- **FR-002**: Archive preserves user-owned financial records, their original relationships and states, and relevant system category context as read-only snapshots before clearing live financial records.
- **FR-003**: Delete removes live financial records and every prior snapshot, while retaining user credentials, profile, session, and notification preferences.
- **FR-004**: Delete requires server-validated current password. Five failed password checks within 15 minutes temporarily block further checks.
- **FR-005**: Both actions clear derived financial notifications and mutation replay records; existing global category defaults remain available for a fresh workspace.
- **FR-006**: Archive list and paginated records are available only to their owner. Archive records have no mutation endpoint.
- **FR-007**: The UI provides clear confirmation and error feedback, clears password fields on close, and reloads workspace state after success.

## Success Criteria

- Archive and delete integration tests cover linked accounts, transactions, transfers, budgets, cards, recurring activity, and goals without violating foreign keys or affecting another user.
- A wrong password changes no records; valid deletion removes current data and archives, then the same session remains usable.
- Failed password attempts accumulate against the five-attempt limit with the default database-backed cache.
- Browser flow covers archive viewing, permanent-deletion warning, wrong-password feedback, and success at 320 px width.

## Assumptions

- “All data” means financial records. Authentication data, profile, and notification preferences remain.
- Multiple archives may exist; each is view-only. Restoring archived data is outside this feature.
- The warning promises no in-app restore; it does not change existing infrastructure backup retention.
