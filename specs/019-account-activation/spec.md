# Feature Specification: Account Activation

- **Feature Branch**: `019-account-activation`
- **Backend Branch**: `019-account-activation` (`../zunera-backend`)
- **Frontend Branch**: `019-account-activation` (`../zunera-frontend`)
- **Status**: Implemented
- **Input**: New accounts begin inactive. Email carries a link to activate the
  account before sign-in.

## User Scenarios & Testing

### User Story 1 - Activate a new account (Priority: P1)

A visitor creates an account, receives an activation email, activates the account,
then signs in to reach the dashboard. Registration and activation create no session.

**Independent Test**: Register a new email, confirm signed-out state, open the
latest emailed link within 24 hours, then sign in with the chosen credentials.

**Acceptance Scenarios**:

1. **Given** valid registration details, **When** the visitor registers, **Then**
   one inactive account is created, a localized activation email is queued, the
   response is `201 {message, activation_required: true}`, and no session starts.
2. **Given** the newest unused activation link is less than 24 hours old,
   **When** the visitor opens it, **Then** the account becomes active, the token
   is consumed, and the visitor sees a sign-in path without being signed in.
3. **Given** an inactive account, **When** correct credentials or an existing
   authenticated session access protected data, **Then** access is denied with
   `account_inactive`; wrong credentials retain generic invalid feedback.

### User Story 2 - Recover an activation link (Priority: P1)

A visitor whose link expired, was replaced, or never arrived can request another
link without revealing whether an email belongs to an account.

**Independent Test**: Resend for inactive, active, and unknown emails; compare
visible responses, mail delivery, send limits, and validity of old and new links.

**Acceptance Scenarios**:

1. **Given** an inactive account, **When** a visitor resends within the limit,
   **Then** a new 24-hour link replaces the previous one.
2. **Given** a valid email for an unknown or active account or a limited resend,
   **When** the visitor requests a link, **Then** the same neutral `202` response
   is shown and no account-specific information is revealed.
3. **Given** an expired, used, replaced, malformed, or unknown link, **When**
   activation is attempted, **Then** account state does not change and an
   inactive visitor can request another link.

### User Story 3 - Preserve existing access (Priority: P1)

Existing users continue to sign in after account activation is introduced.

**Independent Test**: Migrate an existing account with empty
`email_verified_at`, then verify that its credentials and protected access work.

**Acceptance Scenario**:

1. **Given** an account created before the activation rollout, **When** the
   migration runs, **Then** its activation timestamp is backfilled and its
   existing access remains available.

### Edge Cases

- A link expires at the 24-hour boundary; opening it then returns
  `activation_link_expired` and does not activate the account.
- A reused, replaced, mismatched, or unknown token returns
  `activation_link_invalid`; malformed email or token input receives field
  validation errors.
- Repeated resend attempts return the same neutral result while mail is limited
  to three sends per address and 20 per IP per hour.
- An inactive account with a session established outside ordinary login cannot
  read any protected API route.
- A delayed or failed mail delivery does not create an active account or a
  session; the visitor retains a resend path.

## Requirements

### Functional Requirements

- **FR-001**: Registration MUST create an inactive account with
  `email_verified_at = null`, return `201` with `activation_required: true`, and
  leave the visitor signed out.
- **FR-002**: The system MUST queue a localized activation email after the
  registration transaction commits. The email MUST explain inactivity, provide
  an activation action, state the 24-hour expiry, and advise recipients who did
  not register.
- **FR-003**: The system MUST keep at most one hashed activation token per user.
  A new send MUST replace the prior token; the newest matching token MUST work
  once before 24 hours and be consumed on confirmation.
- **FR-004**: Confirmation MUST activate the account without starting a session.
  Expired links MUST use `activation_link_expired`; used, replaced, mismatched,
  and unknown links MUST use `activation_link_invalid`.
- **FR-005**: Correct credentials for an inactive account MUST return `403` with
  `account_inactive`. Wrong credentials MUST retain the generic `401` outcome.
  Every protected API route MUST also deny an inactive authenticated account.
- **FR-006**: Resend MUST return neutral `202` feedback for every valid email,
  regardless of account existence, state, or send limit. Only inactive accounts
  MAY receive a new link, limited to three sends per address and 20 per IP per
  hour.
- **FR-007**: The migration MUST mark all existing accounts active before the
  new registration behavior is served, including accounts with an empty prior
  `email_verified_at`.
- **FR-008**: The UI MUST show registration confirmation, activation result,
  resend, and inactive sign-in guidance in PT-BR and English. It MUST remove the
  token from the browser URL before posting it and offer manual sign-in after
  successful activation.
- **FR-009**: Registration validation, email normalization, password rules,
  CSRF handling, and login throttling MUST remain consistent with user auth.

### Security and Quality Requirements

- **SQR-001**: Confirm and resend MUST validate email and token input on the
  backend. The activation table MUST store only a token hash; raw tokens MUST
  stay out of logs and API responses. Queued mail delivery carries the raw token
  until the activation email is sent.
- **SQR-002**: Resend feedback MUST avoid account enumeration. Correct inactive
  credentials may show activation guidance, but incorrect credentials MUST stay
  generic. Rate-limit keys MUST not contain recoverable email content.
- **SQR-003**: Backend feature and notification tests MUST cover registration,
  token lifetime, replacement, single use, resend limits, existing-user
  migration, and protected denial. Frontend unit and browser tests MUST cover
  the register-to-activate journey, localization, signed-out state, and token
  removal from the URL.
- **SQR-004**: No new uploads or secrets are required. The queued email MUST
  have branded HTML and plain-text representations. Authentication screens MUST
  remain keyboard usable at 320px width and 200% zoom.

### Delivery Scope

1. **Backend** (`../zunera-backend`): Define the activation API contract, apply
   the existing-user migration, validate requests, implement token and account
   services, enforce active-account authorization, queue mail, and test outcomes.
2. **Frontend** (`../zunera-frontend`): Consume the completed API contract in the
   auth service and store; present confirmation, activation, resend, and sign-in
   guidance; localize and test the UI.

### Key Entities

- **User account**: `email_verified_at` records activation; `null` means a new
  account cannot sign in or access protected data.
- **Account activation token**: One hashed token and issue time per inactive
  user, deleted on successful confirmation or user deletion.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Every tested new registration creates one inactive account,
  returns `activation_required: true`, and creates no session.
- **SC-002**: Every tested latest valid link activates exactly once before 24
  hours; every tested expired, reused, replaced, malformed, and unknown link
  leaves account state unchanged.
- **SC-003**: Every tested valid-email resend returns identical public status
  and wording for inactive, active, unknown, and limited cases; sends remain
  within the per-address and per-IP hourly limits.
- **SC-004**: Every tested pre-rollout account retains sign-in and protected
  access after migration; every tested inactive account is denied on login and
  protected API routes.
- **SC-005**: The critical register-to-activate-to-sign-in journey works in both
  supported languages, by keyboard, at 320px width and 200% zoom; the raw token
  is absent from the browser URL before the confirmation request.

## Assumptions

- Email remains the only activation channel and password remains the sign-in
  method. Activation does not automatically authenticate.
- The migration runs before the new backend registration flow is served.
- An `auth-mail` queue worker runs after deployment so queued activation mail
  can be delivered.
