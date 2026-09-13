# Feature Specification: Account Transfers

**Feature Branch**: `005-account-transfers`  
**Backend Branch**: `005-account-transfers` (`../zunera-backend`)  
**Frontend Branch**: `005-account-transfers` (`../zunera-frontend`)  
**Created**: 2026-09-13  
**Status**: Ready for implementation  
**Input**: User description: "Create the Transfers feature for Zunera."

## Clarifications

### Session 2026-09-13

- Q: How should Zunera handle a transfer that would make the source balance
  negative? → A: Reject it for every account. A pending transfer may be recorded,
  but it must be rejected whenever it would become effective while insufficient
  funds remain.
- Q: Where should transfers appear in financial history? → A: Include them in
  existing financial and transaction history, clearly labelled separately from
  income and expenses.
- Q: Should a pending transfer reserve source funds? → A: No. A pending transfer
  changes neither current nor available balance; source funds are checked only
  when it becomes effective.

### Compatibility decisions

- Transfers use established financial lifecycle: `pending` for planned or
  not-yet-effective movement and `effective` for an occurred movement. Only an
  effective transfer changes balances.
- A past- or current-dated transfer defaults to effective. A future-dated
  transfer defaults to pending and does not affect current balances unless user
  later explicitly marks it effective.
- Removal is recoverable lifecycle action, not permanent deletion. A removed
  transfer is excluded from current balances and normal history, remains
  available to its owner in a Removed view, and can be restored.
- Transfers have own history and details and also appear in existing financial
  and transaction history. They remain clearly recognizable as transfers, never
  become income or expense transactions, and never contribute to income, expense,
  financial result, net-worth, or income-versus-expense reporting.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Record an Account Transfer (Priority: P1)

As an authenticated user, I want to move money between two of my active
financial accounts, so that my account balances reflect where my money is held
without changing my overall money.

**Why this priority**: Moving existing money is core value; without it, users
must misclassify transfers as income or expense.

**Independent Test**: A signed-in user with two eligible accounts can record a
valid transfer and see source decrease, destination increase, and combined
balance remain unchanged when transfer is effective.

**Acceptance Scenarios**:

1. **Given** an authenticated user owns two active financial accounts with a
   combined balance of R$ 7.000,00, **When** they record an effective transfer
   of R$ 1.000,00 from account holding R$ 5.000,00 to account holding
   R$ 2.000,00, **Then** first balance is R$ 4.000,00, second is R$ 3.000,00,
   and combined balance remains R$ 7.000,00.
2. **Given** a user has an active source account and destination account,
   **When** they choose same account for both sides, **Then** no transfer is
   recorded and feedback explains that two different accounts are required.
3. **Given** a user chooses an account they do not own for either side,
   **When** they attempt to record a transfer, **Then** it is denied without
   exposing other account's details and neither balance changes.
4. **Given** a user records a transfer dated in future, **When** it is saved,
   **Then** it is pending, is clearly identified as planned, and changes neither
   account's current balance.
5. **Given** a user enters zero, negative amount, malformed Brazilian money, or
   amount with more than two decimal places, **When** they submit transfer,
   **Then** it is not recorded and clear feedback identifies amount problem.
6. **Given** an effective transfer would reduce source account below R$ 0,00,
   **When** user creates, corrects, restores, or makes it effective, **Then**
   action is rejected with insufficient-balance feedback and neither account
   balance changes.
7. **Given** a source account has a pending transfer, **When** user records
   another effective transfer using its available balance, **Then** pending
   transfer does not reserve funds or block action; it is checked only when
   later made effective.

---

### User Story 2 - Review and Find Transfers (Priority: P2)

As an authenticated user, I want to browse, inspect, filter, and search my
transfers, so that I can understand how money moved between my accounts.

**Why this priority**: Transfer must be auditable and discoverable after it is
recorded; otherwise users cannot reconcile balances or correct mistakes.

**Independent Test**: A signed-in user with transfers across accounts, dates,
statuses, and optional text can find one transfer, open detail, and confirm both
sides of movement.

**Acceptance Scenarios**:

1. **Given** a user has recorded transfers, **When** they open transfer
   history, **Then** they see own transfers with source, destination, amount,
   date, status, and description or notes when present.
2. **Given** a user selects a transfer, **When** its details open, **Then** two
   associated accounts, amount, date, status, optional text, and removed state
   when relevant are shown as one financial event.
3. **Given** a user applies a date range, source-account, destination-account,
   and status filter, **When** history is displayed, **Then** it contains only
   transfers matching every selected filter and identifies active filters.
4. **Given** a user enters text from transfer description or notes, **When**
   they search, **Then** matching transfers are found without case or accent
   differences preventing a match.
5. **Given** an account on an existing transfer has later been archived,
   **When** user views that transfer, **Then** both historical account identities
   remain visible and archived account is clearly labelled.
6. **Given** another user has transfers, **When** user opens history, searches,
   filters, or requests detail, **Then** no other user's transfer or account
   information is shown.

---

### User Story 3 - Correct, Remove, and Restore a Transfer (Priority: P2)

As an authenticated user, I want to correct or recover my transfers, so that
mistakes do not leave account balances or history misleading.

**Why this priority**: Transfers affect two balances at once; safe correction
and recovery retain trust in financial record.

**Independent Test**: A signed-in user can update every permitted transfer
field, remove an effective transfer, and restore it while each outcome keeps
both account balances mutually consistent.

**Acceptance Scenarios**:

1. **Given** a user owns an effective transfer, **When** they validly change its
   accounts, amount, date, or status, **Then** prior effect is fully replaced by
   new effect across both affected accounts and no account has duplicated, stale,
   or missing amount.
2. **Given** a user owns an effective transfer, **When** they remove it,
   **Then** source account receives amount back, destination account loses that
   amount, it no longer affects current balances or normal history, and removal
   feedback is shown.
3. **Given** a user owns a removed transfer, **When** they restore it with valid
   status, **Then** it regains historical information and affects both balances
   exactly once only if restored as effective.
4. **Given** an existing transfer references an account now archived, **When**
   user corrects amount, date, optional text, or status while retaining
   association, **Then** historical account remains visible but cannot be newly
   selected as source or destination.
5. **Given** a user attempts to change, remove, or restore another user's
   transfer, **When** action is attempted, **Then** it is denied without
   revealing transfer or account details.
6. **Given** attempted change results in invalid accounts, amount, date, or
   status, **When** submitted, **Then** original transfer and both balances
   remain unchanged and corrective feedback is shown.

---

### User Story 4 - Distinguish Transfers from Income and Expenses (Priority: P3)

As an authenticated user, I want transfers to be visibly distinct from income
and expenses in my financial history, so that moving my own money does not
misstate my financial performance.

**Why this priority**: Clear classification prevents false spending and income
interpretations while preserving one understandable financial history.

**Independent Test**: A user can identify transfer in mixed financial history
without relying on color and verify reports retain income, expense, result, and
net-worth values after transfer.

**Acceptance Scenarios**:

1. **Given** a user has income, expense, and transfer movements, **When** they
   view existing financial or transaction history, **Then** transfer appears
   alongside those movements and is labelled and described as account-to-account
   movement rather than income or expense.
2. **Given** an effective transfer, **When** financial totals or
   income-versus-expense report is viewed before and after it, **Then** total
   income, total expenses, financial result, and net worth do not change solely
   because of that transfer.
3. **Given** Light, Dark, or System theme, narrow screen, keyboard-only
   navigation, assistive technology, or high zoom, **When** user completes
   primary transfer journey, **Then** transfer information and actions remain
   understandable and usable without color being sole distinction.

### Edge Cases

- User has fewer than two active financial accounts, so no new transfer can be
  recorded.
- User tries to use same account as source and destination, including by editing
  existing transfer.
- Either selected account is archived, inactive, unavailable, or belongs to
  another user.
- Existing transfer references account later archived, restored, renamed, or
  otherwise changed.
- Effective transfer changes source, destination, amount, date, status, or more
  than one of these values in one correction.
- Pending transfer becomes effective; effective transfer becomes pending; or same
  lifecycle action is retried after uncertain feedback.
- Pending transfer does not reserve source funds. It may later be rejected when
  becoming effective if another effective movement has used those funds.
- Future-dated transfer is explicitly marked effective, or current/past
  effective transfer is redated into future.
- An effective transfer would reduce source balance below zero; it is rejected,
  while a pending transfer may remain planned until funds are available.
- User submits Brazilian amount with currency symbol, thousands separator, comma
  decimal separator, spaces, negative sign, zero, or excessive fractional
  precision.
- Date is missing, malformed, outside supported range, far in past, or future.
- Double-click, retry, or delayed response repeats create, update, remove,
  restore, or status-change request.
- User searches with empty, case-varied, accent-varied, or no-match text;
  combines filters; or has enough history to require progressive loading.
- User tries to permanently delete transfer although lifecycle supports Remove,
  Removed, and Restore instead.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow authenticated users to create, view, search,
  filter, edit, remove, and restore only transfers that involve financial
  accounts they own.
- **FR-002**: A transfer MUST represent one movement between exactly one source
  account and exactly one destination account. Its two sides MUST always be
  preserved and presented as one financial event.
- **FR-003**: A transfer MUST have source account, destination account, positive
  monetary amount, and transfer date. It MAY have optional description and
  optional notes.
- **FR-004**: Source and destination accounts MUST be different. Transfer with
  identical accounts MUST be rejected without changing balance.
- **FR-005**: Both accounts selected for new transfer or changed association MUST
  be active, available financial accounts owned by authenticated user. Another
  user's account MUST be denied without revealing details.
- **FR-006**: Account later archived or inactive MUST remain historically
  identifiable on existing transfer. It MUST not be normal selectable account
  for new transfer or new association, but existing association MAY be retained
  while other permitted transfer fields are edited.
- **FR-007**: Transfer amounts MUST be positive values from R$ 0,01 through
  R$ 999.999.999,99, with at most two decimal places. Zero, negative,
  malformed, over-precision, and out-of-range amounts MUST be rejected with
  actionable feedback.
- **FR-008**: Monetary input, display, aggregation, and balance adjustment MUST
  support Brazilian currency conventions and preserve exact cents without
  changing intended amount.
- **FR-009**: Transfer date MUST represent when movement occurred or is expected
  to occur. Past dates are supported; missing, malformed, and out-of-range dates
  MUST be rejected with actionable feedback.
- **FR-010**: Transfer dates from 1900-01-01 through 2100-12-31 inclusive MUST
  be supported. Date input and display MUST accept Brazilian conventions while
  keeping intended calendar date unambiguous.
- **FR-011**: Each transfer MUST have exactly one financial status: `pending`
  or `effective`. Pending means planned or not yet financially occurred;
  effective means financially occurred.
- **FR-012**: Only effective, non-removed transfer MUST affect account balances.
  Pending, non-removed transfer and every removed transfer MUST not affect
  current or available balances. A pending transfer MUST NOT reserve source
  funds; funds are checked only when it becomes effective.
- **FR-013**: Transfer dated today or past MUST default to effective.
  Future-dated transfer MUST default to pending and stay out of current balances
  until user explicitly marks it effective. Changing status MUST apply or reverse
  both transfer sides exactly once.
- **FR-014**: When transfer becomes effective, source balance MUST decrease by
  amount and destination balance MUST increase by same amount. When it ceases to
  be effective, both effects MUST be reversed.
- **FR-015**: Every create, correction, status change, removal, restore, and
  repeated attempt MUST preserve user's combined balance across all accounts
  affected by transfer. No completed action may leave only one side applied.
- **FR-016**: System MUST reject an effective transfer creation, correction,
  restoration, or status change when it would reduce source account balance below
  R$ 0,00. A pending transfer MAY be recorded, but it MUST be rejected whenever
  it would become effective while source funds remain insufficient.
- **FR-017**: System MUST allow owner to correct non-removed transfer's source
  account, destination account, amount, date, financial status, description,
  and notes when resulting transfer is valid. Calculation-affecting changes MUST
  replace prior complete financial effect with new complete effect.
- **FR-018**: Rejected create, correction, removal, restoration, or status
  change MUST leave transfer and every affected account balance exactly as they
  were before attempt.
- **FR-019**: Removing transfer MUST be recoverable lifecycle change, not
  permanent deletion. Removal of effective transfer MUST reverse both balance
  effects. Removed transfer MUST retain historical data, be excluded from normal
  history and current balances, remain visible to owner as Removed, and be
  restorable.
- **FR-020**: Restoring transfer MUST retain recorded source, destination,
  amount, date, and optional text. It MUST affect both balances exactly once only
  when restored as effective and must follow status and date rules for new
  transfer.
- **FR-021**: Users MUST be able to view own transfer history and full detail of
  any transfer. Normal history MUST order recent financial activity first and
  clearly show source, destination, amount, date, status, and optional text.
- **FR-022**: Users MUST be able to filter own transfers by date or date range,
  source account, destination account, and transfer status. Multiple filters MUST
  combine, remain visible, and be clearable.
- **FR-023**: Users MUST be able to search own transfers by meaningful text in
  description and notes. Search MUST ignore case and accent differences and
  combine predictably with active filters.
- **FR-024**: Transfer history MUST support users with at least 5,000 transfers,
  show matching results in progressive batches of no more than 50, and make total
  matching count visible.
- **FR-025**: Transfers MUST remain distinct from income and expenses in every
  user-facing history, summary, and report. Transfer MUST NOT change total income,
  total expenses, financial result, net worth, or income-versus-expense report
  merely because money moved between owned accounts.
- **FR-026**: Existing financial and transaction history MUST include transfers
  alongside income and expense movements, identify each transfer as an
  account-to-account movement, and distinguish it from income and expenses
  without relying only on color.
- **FR-027**: System MUST prevent duplicate financial effects from repeated or
  retried create, edit, remove, restore, and status-change actions, while
  allowing intentionally similar transfers recorded as separate financial events.
- **FR-028**: Every successful create, update, removal, and restoration MUST
  provide clear confirmation. Invalid amount, account, date, or status values;
  identical accounts; unavailable accounts; unauthorized actions; and blocked
  lifecycle actions MUST provide clear, privacy-safe correction guidance.
- **FR-029**: Transfer experiences MUST follow Zunera Design Foundation,
  Application Shell, Navigation, and shared-component guidance. They MUST remain
  usable in Light, Dark, and System themes, supported screen sizes, keyboard-only
  navigation, assistive technology, and high zoom.
- **FR-030**: Feature MUST NOT support transfers to another Zunera user,
  external-bank or PIX initiation, bank synchronization, automatic
  reconciliation, currency conversion, different currencies, recurring
  transfers, investment transactions, or credit-card payments.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Authorization and input validation MUST cover every transfer
  action, including source and destination ownership, account lifecycle, amount,
  date, status, removal, restoration, filters, search, and direct access.
- **SQR-002**: Automated coverage MUST verify transfer behavior, ownership
  denials, validation feedback, both-account balance consistency, status
  transitions, correction, removal, restoration, archived-account history,
  filtering, search, reporting exclusion, and critical user journeys.
- **SQR-003**: Transfer data contains financial history and MUST be visible only
  to its owner. Feature does not collect uploads or require new secrets.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define protected transfer behavior,
   authorization, validation, financial consistency, historical lifecycle,
   reporting distinction, and automated verification before user interface work
   begins.
2. **Frontend** (`../zunera-frontend`): After backend behavior is confirmed,
   provide transfer history, filters and search, create and edit flows, details,
   Removed recovery, feedback, and accessible responsive presentation that
   consume confirmed behavior.

### Key Entities *(include if feature involves data)*

- **Transfer**: One owned movement between source account and destination
  account, including amount, date, financial status, optional description and
  notes, and removal state.
- **Financial Account**: Owned account that can be transfer source or
  destination while active; it retains historical identity when archived.
- **Transfer Financial Status**: Pending or effective state that determines
  whether non-removed transfer affects both account balances.
- **Financial History Entry**: User-facing representation of income, expense, or
  transfer movement that keeps classification understandable.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of acceptance-test users with two eligible accounts
  can record valid transfer in under one minute without assistance.
- **SC-002**: In balance-consistency validation, 100% of effective transfer
  creation, correction, status change, removal, restoration, and repeated-action
  scenarios preserve combined balance and apply equal opposite changes to both
  accounts.
- **SC-003**: In authorization validation, 100% of attempts to view, create,
  edit, remove, restore, filter by, search for, or associate another user's
  transfer or financial account are denied without revealing data.
- **SC-004**: At least 90% of acceptance-test users can locate specified transfer
  by date, account, status, or text search in under 30 seconds from history with
  at least 50 transfers.
- **SC-005**: In reporting validation, 100% of effective transfer scenarios leave
  total income, total expenses, financial result, net worth, and
  income-versus-expense reports unchanged solely because of transfer.
- **SC-006**: In historical-integrity validation, 100% of transfers involving
  subsequently archived accounts remain understandable and show both original
  account identities.
- **SC-007**: Users with 5,000 transfers see first history batch within two
  seconds, and 100% of matching transfers remain reachable through progressive
  loading, filters, and search.
- **SC-008**: In responsive, theme, and accessibility validation, 100% of
  primary transfer journeys can be completed in supported layouts, Light, Dark,
  and System themes, and with keyboard-only navigation; transfer classification
  remains understandable without color alone.

## Assumptions

- Users already have authenticated Zunera accounts and financial accounts.
  Transfer ownership and account lifecycle behavior build on Financial Accounts
  feature rather than redefining it.
- Inactive account in this feature means archived account as defined by Financial
  Accounts. Existing transfer history stays accessible after archival; only new
  or changed associations require active accounts.
- Transfer dates use established supported range of 1900-01-01 through
  2100-12-31. Future date is valid but defaults to pending.
- Transfers use exactly two financial statuses, pending and effective, matching
  Transactions feature. Only effective non-removed transfers change current
  balances; explicit user status change is required to make future-dated transfer
  effective.
- Remove, Removed, and Restore are user-facing lifecycle terms, matching
  Transactions feature. Permanent deletion is excluded to preserve financial
  history and permit recovery from accidental removal.
- Transfers are shown in dedicated transfer history and in existing financial and
  transaction history. They remain distinct movement class and do not reclassify
  existing income or expense records.
- Brazilian Real is only currency. One transfer always moves same Brazilian Real
  amount out of source and into destination.
- Description and notes are independent optional text fields. When present,
  descriptions contain at most 200 user-visible characters and notes at most
  1,000; blank whitespace-only values are not retained.
- Source accounts must never become negative because of a transfer. A pending
  transfer remains planned without reserving funds and is checked only when its
  source is made effective.
