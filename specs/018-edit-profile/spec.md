# Feature Specification: Edit Profile

**Feature Branch**: `018-edit-profile`  
**Backend Branch**: `018-edit-profile` (`../zunera-backend`)  
**Frontend Branch**: `018-edit-profile` (`../zunera-frontend`)  
**Created**: 2026-10-01  
**Status**: Implemented  
**Input**: Add separate Edit profile and Change password account-menu actions; edit name, show email without allowing changes, and change password independently.

## User Scenarios & Testing

### User Story 1 - Edit name (Priority: P1)

A signed-in user opens Edit profile from the account menu and changes their displayed name without leaving the current page.

**Independent Test**: Change name, then confirm updated name in header and after a reload.

**Acceptance Scenarios**:

1. **Given** an active session, **When** the user opens Edit profile, **Then** the dialog shows current name and email.
2. **Given** a valid new name, **When** the user saves, **Then** only their name changes and the header updates immediately.
3. **Given** an invalid name, **When** the user saves, **Then** the dialog shows an error and stored name stays unchanged.

### User Story 2 - Change password (Priority: P2)

A signed-in user opens Change password from the account menu and changes their password in its own dialog.

**Independent Test**: Change password, then confirm the new password signs in and the old password does not.

**Acceptance Scenarios**:

1. **Given** a correct current password and safe matching new password, **When** the user saves, **Then** their password changes, this session stays active, and other sessions end.
2. **Given** an incorrect current password or invalid new password, **When** the user saves, **Then** their password remains unchanged and an actionable error appears.
3. **Given** a successful change, **When** an outstanding password-recovery link is used, **Then** it no longer works.

### Edge Cases

- Email is visible for reference but cannot be edited through either the dialog or profile action.
- Repeated submit does not produce duplicate changes or notices.
- After five failed current-password attempts in 15 minutes, further password-change attempts are temporarily limited.
- Closing and reopening the password dialog clears sensitive fields and validation errors.
- Expired sessions cannot save a profile or password change.

## Requirements

### Functional Requirements

- **FR-001**: Account menu MUST include Edit profile and Change password above language choices; each opens its own dialog without navigation.
- **FR-002**: Dialog MUST show current name in an editable field and current email in a disabled field.
- **FR-003**: Signed-in user MUST be able to update only their name using the registration name rules.
- **FR-004**: Email MUST remain unchanged even if a client submits an email field to the profile action.
- **FR-005**: Change password dialog MUST provide a form requiring current password, safe new password, and matching confirmation. Edit profile dialog MUST contain no password fields.
- **FR-006**: Successful password change MUST keep current session, end other sessions, invalidate recovery tokens, and send existing password-change notice.
- **FR-007**: Failed current-password attempts MUST be limited to five per 15 minutes across the password action.
- **FR-008**: Name and password dialogs MUST show independent success and error feedback in Portuguese and English.

### Security and Quality Requirements

- **SQR-001**: Both mutations MUST require authenticated, unexpired session; validate input server-side; and never expose passwords in responses or logs.
- **SQR-002**: Backend feature/contract tests and frontend service/component tests MUST cover success, failures, and session behavior. A browser test MUST cover the critical dialog-to-API journey.
- **SQR-003**: No new secrets, packages, uploads, or database tables are required. Password notices use the existing queued auth mail setup.

### Delivery Scope

1. **Backend** (`../zunera-backend`): Protected name and password API contracts, validation, service mutations, session revocation, notice, and tests.
2. **Frontend** (`../zunera-frontend`): Two account-menu entries and localized dialogs, auth service/store updates, and tests consuming the backend contract.

### Key Entities

- **User**: Owns name, immutable-in-this-feature email, and password hash.
- **Session**: Current session remains after password change; other sessions end.
- **Recovery token**: Outstanding tokens become invalid after password change.

## Success Criteria

### Measurable Outcomes

- **SC-001**: In acceptance flows, every valid name change appears in header immediately and persists after reload.
- **SC-002**: In acceptance flows, every invalid or unauthorized request leaves account data unchanged.
- **SC-003**: In acceptance flows, a successful password change keeps only the initiating session active and prevents reuse of old password or recovery tokens.
- **SC-004**: Both forms remain usable with keyboard, at 320px width and 200% zoom, in Portuguese and English.

## Assumptions

- Email remains visible but is not editable; email change notices and verification are out of scope.
- Name edit needs no password. Password change needs current password.
- Existing database-backed sessions and queued auth mail are available.
