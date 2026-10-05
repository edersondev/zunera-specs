# Feature Specification: Restore Account Data

**Feature Branch**: `021-restore-account-data`

**Backend Branch**: `021-restore-account-data` (`../zunera-backend`)

**Frontend Branch**: `021-restore-account-data` (`../zunera-frontend`)

**Status**: Implemented

**Input**: Restore financial workspace from an archive item on Reset account data page.

## User Scenarios

### Restore an archive

A signed-in user chooses **Restore** on an archive item and confirms. If the current workspace has financial records, Zunera saves them in a new archive first. The selected archive becomes the live workspace and remains available in the list. If current workspace is empty, no empty backup is created. An empty archive restores an empty workspace.

### Failure and access

- Missing or another user's archive cannot be restored or disclosed.
- Invalid archived records, missing system categories, or a database failure leave current data and archive list unchanged.
- Archive detail remains read-only; only the whole archive can be restored from the list.
- Confirmation and errors work in Portuguese and English, by keyboard, and at 320 px width.

## Requirements

- **FR-001**: Only an authenticated archive owner can restore a complete archive. The request accepts no fields and needs no password.
- **FR-002**: Restore saves any existing user-owned financial records as a new archive, clears current financial data, and restores the selected archive atomically.
- **FR-003**: Restored records keep amounts, dates, lifecycle states, historical snapshots, and relationships. New live IDs are assigned and references remapped; archived copies remain immutable.
- **FR-004**: Relevant system category references resolve to existing global defaults without modifying them. An unresolved reference blocks the entire restore.
- **FR-005**: Stale derived notifications and mutation replay records are cleared; applicable notification projections are rebuilt. Profile, credentials, session, and notification preferences remain.
- **FR-006**: The archive list offers a labeled Restore action with confirmation, loading, success navigation, and recoverable error feedback.

## Success Criteria

- Backend round trip restores linked accounts, categories, transactions, transfers, budgets, cards, recurring records, and goals; unrelated users stay unchanged.
- Restoring into a populated workspace creates one new archive. Restoring into an empty workspace creates none. The selected archive remains available, including after repeated restores.
- Corrupt snapshots roll back without losing current data or creating a backup.
- Browser flow covers confirmation, error, success, retained archives, and 320 px layout.
