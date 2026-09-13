# Feature Specification: Transaction Management

**Feature Branch**: `004-transaction-management`
**Backend Branch**: `004-transaction-management` (`../zunera-backend`)
**Frontend Branch**: `004-transaction-management` (`../zunera-frontend`)
**Created**: 2026-09-11
**Status**: Draft
**Input**: User description: "Create the Transactions feature for Zunera. The
system must allow authenticated users to record, manage, and track the financial
movements that affect their financial accounts."

## Clarifications

### Session 2026-09-11

- Q: Which transaction states exist, and which are financially effective? → A:
  Two states: pending (planned) and effective (settled). Only effective
  transactions affect account balances. Newly recorded transactions default to
  effective unless the transaction date is in the future.
- Q: Does removing a transaction permanently delete it or retain it in a
  non-counting state? → A: Removal is a lifecycle change. The transaction is
  kept in a removed state, out of balances and the normal history, and remains
  restorable while historical information stays intact.
- Q: Must future-dated transactions be recorded as planned, or may they be
  financially effective? → A: Future-dated transactions are recorded as pending
  and do not affect balances until they are marked effective.
- Q: What are the supported monetary amount and transaction date bounds? → A:
  Amount from R$ 0.01 up to R$ 999,999,999.99, and transaction date from
  1900-01-01 through 2100-12-31.
- Q: What does transaction search match against? → A: The transaction
  description and the optional note. Category, account, type, and status are
  handled through filters instead.
- Q: How are transactions tied to an archived account or category edited? → A:
  Every field stays editable while that archived account or category keeps its
  existing association. Archived items appear as read-only labels and are never
  selectable for a new or different association.
- Q: How is a large transaction history loaded? → A: Newest first, in
  progressive batches of at most 50 transactions, with the number of matching
  transactions shown and support for at least 5,000 transactions per user.
- Q: Which term does the interface use for a transaction that is out of the
  active history? → A: The action is Remove, the state is Removed, and the
  recovery action is Restore, available from a removed-transactions view.
- Q: What happens when an effective transaction's date is edited into the
  future? → A: The effective state is preserved, it keeps affecting the
  balance, and the user receives a clear notice with the option to set it
  pending. Nothing changes silently.
- Q: Can a removed transaction be edited before it is restored? → A: No.
  Editing a removed transaction is rejected with a state conflict that tells the
  user to restore it first; every field becomes editable again after restore.
- Q: What happens to an account's or category's pending transactions when it is
  archived? → A: Archiving proceeds. Pending transactions stay pending, keep no
  balance effect, and remain visible and editable while keeping that archived
  association.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Record Income and Expense Transactions (Priority: P1)

As an authenticated user, I want to record money entering or leaving one of my
financial accounts, so that my financial history reflects what actually happened
with my money.

**Why this priority**: Recording transactions is the core purpose of the
feature. Without it, no other transaction capability delivers value.

**Independent Test**: A signed-in user with at least one active financial
account and one applicable category can record an income transaction and an
expense transaction, see each one in their transaction list with description,
amount, type, date, account, and category, and receive clear success feedback.

**Acceptance Scenarios**:

1. **Given** an authenticated user has an active financial account and an
   income category available to them, **When** they record an income
   transaction with a description, a positive amount, and a transaction date,
   **Then** the transaction is stored, shown in their transaction history as
   income, and success feedback is shown.
2. **Given** an authenticated user has an active financial account and an
   expense category available to them, **When** they record an expense
   transaction with a description, a positive amount, and a transaction date,
   **Then** the transaction is stored, shown in their transaction history as an
   expense, and success feedback is shown.
3. **Given** an authenticated user records a transaction with an additional
   note, **When** the transaction is saved and later opened, **Then** the note
   is preserved and displayed with the transaction details.
4. **Given** an authenticated user provides an unusable description, a
   non-positive or malformed amount, an invalid date, or no account, category,
   or type, **When** they attempt to record the transaction, **Then** no
   transaction is created and clear, field-relevant correction feedback is
   shown.
5. **Given** an unauthenticated visitor attempts to record or view
   transactions, **When** access is requested, **Then** access is denied
   without exposing any transaction information.

---

### User Story 2 - Keep Account Balances Accurate (Priority: P1)

As an authenticated user, I want my financial account balances to stay correct
whenever my transactions change, so that I can trust the balance I see for
everyday financial decisions.

**Why this priority**: Balance integrity is the financial promise of the
feature. A transaction that does not correctly change the affected balance is
worse than no transaction at all.

**Independent Test**: Starting from a known account balance, a signed-in user
records an income and an expense, then edits the amount, changes the type,
moves the transaction to another account, changes its financial status, and
removes it, confirming after each step that the affected account balances match
the expected arithmetic.

**Acceptance Scenarios**:

1. **Given** an account with a known starting balance, **When** the user records
   a financially effective income transaction of a given amount, **Then** the
   account balance increases by exactly that amount.
2. **Given** an account with a known starting balance, **When** the user records
   a financially effective expense transaction of a given amount, **Then** the
   account balance decreases by exactly that amount.
3. **Given** a transaction already counted in a balance, **When** the user
   changes its amount, type, account, date, or financial status, **Then** the
   previously affected balance and the newly affected balance are each restated
   so that no account is left with a stale or duplicated effect.
4. **Given** a transaction is removed, **When** the removal completes, **Then**
   its effect on the associated account balance is fully reversed.
5. **Given** an account created by the Financial Accounts feature with an
   initial balance, **When** transactions are recorded against it, **Then** the
   current balance remains consistent with that initial balance plus all
   financially effective movements.

---

### User Story 3 - View and Inspect Transaction History (Priority: P2)

As an authenticated user, I want to review my transactions and inspect the
details of a single transaction, so that I can understand and verify my
financial activity.

**Why this priority**: Users need to confirm what was recorded and reconcile it
against reality, which also drives corrections and removals.

**Independent Test**: A signed-in user with several recorded transactions can
open their history, see entries ordered with the most recent financial activity
easiest to identify, distinguish income from expense without relying on color
alone, and open one transaction to see all of its details.

**Acceptance Scenarios**:

1. **Given** an authenticated user has recorded transactions, **When** they view
   their transaction history, **Then** each entry communicates description,
   amount, type, date, account, category, and status when relevant.
2. **Given** a user with transactions on several dates, **When** they open their
   history, **Then** entries are presented in chronological order with the most
   recent financial activity easy to identify.
3. **Given** a mixed list of income and expense entries, **When** it is
   displayed, **Then** income and expense are visually distinguishable by more
   than color alone, following the Zunera Design Foundation financial semantic
   color rules.
4. **Given** an authenticated user selects a transaction from their list,
   **When** its details open, **Then** all recorded information for that
   transaction is shown, including note and status when present.
5. **Given** an authenticated user has no transactions, **When** they open their
   history, **Then** an empty state explains that no transactions exist yet and
   invites them to record the first one.
6. **Given** two users each have transactions, **When** one user views their
   history or a transaction's details, **Then** only their own transactions are
   ever shown.
7. **Given** a user has more transactions than fit in one batch, **When** they
   open their history, **Then** the most recent transactions appear first in
   batches of at most 50, further transactions become available progressively,
   and the number of matching transactions is shown.

---

### User Story 4 - Correct and Manage Existing Transactions (Priority: P2)

As an authenticated user, I want to correct or remove transactions I recorded,
so that mistakes and changing circumstances do not leave my history inaccurate.

**Why this priority**: Real financial records are corrected regularly; without
this, users would be trapped with wrong data and lose trust in balances.

**Independent Test**: A signed-in user can edit the description, amount, type,
account, category, date, status, and note of a transaction they own — including
correcting a transaction in an archived account — and can remove a transaction,
receiving clear feedback for each outcome.

**Acceptance Scenarios**:

1. **Given** an authenticated user owns a transaction, **When** they edit its
   description, note, amount, type, category, account, date, or status with valid
   information, **Then** the change is saved, the financial effect is restated
   correctly, and update feedback is shown.
2. **Given** a transaction is associated with an account or category that has
   since been archived, **When** the user views or corrects descriptive details
   of that transaction, **Then** the historical association and its meaning are
   preserved and the transaction remains accessible.
3. **Given** an authenticated user owns a transaction, **When** they remove it,
   **Then** removal feedback is shown and the transaction no longer appears in
   their normal transaction history.
4. **Given** a user attempts to view, edit, or remove a transaction owned by
   another user, **When** the action is attempted, **Then** it is denied without
   revealing the other user's transaction details.
5. **Given** a user attempts an edit or removal that the current state of the
   transaction, account, or category does not allow, **When** they attempt it,
   **Then** no unintended change occurs and clear state-based feedback explains
   why.
6. **Given** a user submits an invalid amount, date, account, category, type, or
   status change while editing, **When** they save, **Then** no change is applied
   and corrective feedback identifies what must be fixed.
7. **Given** an authenticated user has removed a transaction, **When** they view
   their removed transactions and restore one, **Then** it becomes pending or
   effective under the same rules as a new transaction, its balance effect
   applies only if it is effective, and restore feedback is shown.
8. **Given** a transaction is associated with an account that has since been
   archived, **When** the user edits its amount or financial status while keeping
   that account, **Then** the edit is saved, the archived account is shown as a
   read-only association, and the affected balance is restated correctly.
9. **Given** an effective transaction, **When** the user edits its date into the
   future, **Then** the transaction stays effective and keeps affecting the
   balance, and a clear notice explains the consequence with the option to set
   it pending.

---

### User Story 5 - Find Transactions by Filter and Search (Priority: P3)

As an authenticated user with a growing history, I want to filter and search my
transactions, so that I can quickly locate the movements I care about.

**Why this priority**: Discovery becomes essential as history grows, but it
depends on transactions already existing and being correct.

**Independent Test**: A signed-in user with transactions across multiple dates,
accounts, categories, types, and statuses can narrow the list by each criterion,
combine criteria, and find an entry by text from its description.

**Acceptance Scenarios**:

1. **Given** a user has transactions across several dates, **When** they filter
   by a date range, **Then** only transactions whose transaction date falls in
   that range are listed.
2. **Given** a user has both income and expense transactions, **When** they
   filter by type, **Then** only transactions of the selected type are listed.
3. **Given** a user has transactions in more than one financial account or
   category, **When** they filter by account or category, **Then** only
   transactions matching that selection are listed, including archived accounts
   and categories where historical entries are relevant.
4. **Given** a user has transactions in more than one financial state, **When**
   they filter by status, **Then** only transactions in the selected state are
   listed.
5. **Given** a user enters meaningful text from a transaction description,
   **When** they search, **Then** matching transactions are listed.
6. **Given** a user applies several filters together, **When** the list is
   presented, **Then** only transactions satisfying all applied criteria are
   listed, and the active criteria are visible and can be cleared.
7. **Given** a set of filters or search text matches no transactions, **When**
   results are shown, **Then** a clear empty state indicates that nothing matched
   and allows the criteria to be adjusted.

---

### Edge Cases

- A user has no financial accounts, no active financial accounts, or no
  applicable category, and therefore cannot record a transaction.
- A user's only available categories belong to a classification that does not
  match the selected transaction type.
- A user records a transaction dated in the future, or a transaction dated far
  in the past.
- A user records two transactions that look identical (same description,
  amount, date, account, and category) one after another.
- A user submits the same transaction twice through a repeated submit, retry, or
  refreshed feedback.
- A user edits a transaction's account to one that has already been archived, or
  edits a transaction that already references an archived account or category.
- An account or category with pending transactions is archived while those
  transactions are still pending.
- A user edits a transaction to a state combination that is not allowed, such as
  an income type with an expense-only category, or an invalid status change.
- A user edits an effective transaction's date into the future, or edits a
  pending transaction's date in either direction.
- A user changes a transaction's type, account, or financial status in a way
  that moves balance effect from one account to another or removes it entirely.
- A user removes a transaction that is associated with an archived account or
  category.
- A user tries to edit a transaction that is currently removed, instead of
  restoring it first.
- A user tries to view, edit, remove, filter by, or search another user's
  transactions, accounts, or custom categories.
- A transaction amount is entered with zero value, negative value, more than two
  decimal places, an unparseable value, Brazilian currency formatting such as
  thousands separators, or an amount larger than the supported maximum.
- A transaction date is empty, malformed, or outside the supported date range.
- A description, note, or search term is blank, whitespace only, or overlong.
- Two accounts of the same user show the same balance, one affecting the other
  through an edit that moves a transaction between them.
- Transaction history, detail, filter, and search experiences are used in Light,
  Dark, or System theme; at narrow supported screen sizes; with keyboard-only
  navigation, assistive technology, or high zoom.
- A user's financial history is reviewed in long lists where transaction
  ordering, progressive loading, and total counts may not be visible at once.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow authenticated users to record a transaction
  for one of their own financial accounts with a description, a positive
  monetary amount, a transaction date, a transaction type, a category, and an
  optional note.
- **FR-002**: Each transaction MUST have exactly one transaction type. This
  feature supports income and expense only; the system MUST reject any other
  type with actionable feedback.
- **FR-003**: An income transaction MUST represent money entering the associated
  account. An expense transaction MUST represent money leaving the associated
  account.
- **FR-004**: The transaction model MUST remain extensible so that additional
  financial movement types, such as transfers between accounts, can be
  introduced later without invalidating previously recorded transactions or
  changing their meaning.
- **FR-005**: The system MUST allow an authenticated user to view their own
  transaction history and the details of any single transaction they own.
- **FR-006**: The system MUST present transactions in chronological order, with
  the most recent financial activity easy to identify.
- **FR-007**: Each transaction entry in a list MUST communicate description,
  amount, type, date, account, category, and status when relevant. Income and
  expense MUST be visually distinguishable without relying exclusively on color,
  following the Zunera Design Foundation financial semantic color rules.
- **FR-008**: Every transaction MUST belong to exactly one financial account
  owned by the authenticated user. A user MUST NOT create, edit, filter by, or
  otherwise use an account belonging to another user in any transaction action.
- **FR-009**: Only active financial accounts MUST be selectable when recording
  or reassigning a transaction. Archived accounts MUST NOT be presented as
  normal choices for a new or changed association, although an existing
  association with an archived account MAY be retained as described in FR-044.
- **FR-010**: Transactions associated with an account that is later archived
  MUST remain historically accessible and understandable, showing that
  account's name together with a visible archived-state label that explains why
  it is unavailable for new movements.
- **FR-011**: Every transaction MUST be associated with exactly one category
  whose financial classification matches the transaction type: income
  transactions with income categories, and expense transactions with expense
  categories.
- **FR-012**: A user MUST NOT associate a transaction with a custom category
  belonging to another user. Only categories available to the authenticated user
  MAY be selected.
- **FR-013**: Only active categories MUST be selectable when recording or
  reassigning a transaction. Archived categories MUST NOT be presented as normal
  choices for a new or changed association, although an existing association
  with an archived category MAY be retained as described in FR-044.
- **FR-014**: Transactions associated with a category that is later archived
  MUST preserve that historical association, and its meaning MUST remain
  understandable in history, detail, filters, and search.
- **FR-015**: Transaction amounts MUST always be positive monetary values with
  no sign used to express direction. The transaction type determines the
  financial effect, and the system MUST reject zero, negative, unparseable, or
  over-precision amounts with actionable feedback.
- **FR-016**: Monetary values MUST support Brazilian currency conventions,
  including Brazilian formatting on input and display, and MUST NOT lose
  precision when they are recorded, edited, aggregated, or used to derive an
  account balance.
- **FR-017**: A transaction amount MUST be at least R$ 0.01 and MUST NOT exceed
  R$ 999,999,999.99, expressed with at most two decimal places. The system MUST
  reject amounts outside that range with actionable feedback.
- **FR-018**: Every transaction MUST have a transaction date representing when
  the financial event occurred. The system MUST support recording transactions
  dated in the past and MUST reject missing, malformed, or out-of-range dates
  with actionable feedback.
- **FR-019**: The transaction date MUST fall between 1900-01-01 and 2100-12-31
  inclusive, so that implausible dates cannot be stored, and the system MUST
  communicate that supported range clearly when a date is rejected.
- **FR-020**: The system MUST record a transaction dated in the future as
  pending, and MUST NOT let it affect any account balance until the user marks
  it effective. A future-dated transaction MUST NOT silently become effective,
  and the system MUST make clear that it is planned rather than already
  occurred.
- **FR-021**: The system MUST support exactly two transaction states: pending
  (planned or not yet settled) and effective (the financial movement has
  occurred). Only effective transactions are financially effective for balance
  purposes. A newly recorded transaction MUST default to effective, except when
  its transaction date is in the future, in which case it defaults to pending.
  The current state MUST be visible wherever it affects a user's understanding
  of their financial position.
- **FR-022**: Pending transactions MUST remain fully visible and manageable
  without changing account balances.
- **FR-023**: The system MUST update financial state consistently when a
  transaction's status changes, so that becoming effective applies the movement
  exactly once and ceasing to be effective removes it exactly once.
- **FR-024**: Account balance MUST be derived consistently from the account's
  original balance plus the effect of its financially effective income
  transactions minus the effect of its financially effective expense
  transactions, and MUST reflect the initial balance defined by the Financial
  Accounts feature.
- **FR-025**: Account balance MUST remain consistent when a transaction is
  created, edited, removed, changed between income and expense where allowed,
  moved to another account where allowed, or changed between financially
  effective and non-effective states.
- **FR-026**: The system MUST allow an authenticated user to edit their own
  transaction's description, note, amount, type, category, account, date, and
  status, subject to the type, category, account, date, and status rules in this
  specification.
- **FR-027**: When an edit changes information that affects financial
  calculations, the system MUST restate the affected financial state so that no
  account retains a stale, duplicated, or missing effect.
- **FR-028**: When an edit is rejected because the resulting state is not
  allowed, the transaction MUST remain exactly as it was before the attempt.
- **FR-029**: The system MUST allow an authenticated user to remove a
  transaction they own. Removal MUST reverse the transaction's effect on any
  account balance that included it, and MUST show removal feedback.
- **FR-030**: Removal MUST be a lifecycle change, not permanent deletion. A
  removed transaction MUST be retained in a removed state, MUST be excluded from
  account balances and from the user's normal transaction history, and MUST
  remain restorable by its owner with its historical information intact.
- **FR-031**: The system MUST prevent duplicate transactions caused by repeated
  submissions of the same create, edit, or removal action; a repeated action
  MUST NOT apply a financial effect more than once.
- **FR-032**: The system MUST allow an authenticated user to filter their
  transactions by date or date range, transaction type, financial account,
  category, and transaction status, and MUST allow multiple filters to be
  combined.
- **FR-033**: The system MUST allow an authenticated user to search their own
  transactions by textual content, matched against the transaction description
  and the optional note. Search MUST ignore differences in letter case and
  accents, and MUST combine with any active filters so that only transactions
  satisfying both the search text and the filters are listed. Category, account,
  type, and status selection are handled through filters rather than search text.
- **FR-034**: The system MUST clearly communicate which filters, search terms, or
  date ranges are currently applied to a transaction list and MUST allow them to
  be cleared.
- **FR-035**: A transaction list, detail, filter, or search result MUST only ever
  include transactions belonging to the authenticated user, and MUST NOT reveal
  the existence or content of another user's transactions, accounts, or custom
  categories.
- **FR-036**: A transaction description MUST contain 1–200 user-visible
  characters after trimming and MUST include at least one non-whitespace
  character. An optional note MUST contain at most 1000 user-visible characters
  after trimming. Blank, whitespace-only, or overlong values MUST be rejected
  with actionable feedback.
- **FR-037**: The system MUST reject any transaction action whose account,
  category, type, status, date, or amount combination is not allowed, without
  applying a partial or unintended change.
- **FR-038**: Every successfully completed transaction creation, update, and
  removal MUST show clear success feedback, and every rejected action MUST show
  clear, privacy-safe feedback that helps the user correct the problem.
- **FR-039**: Transaction experiences MUST follow the existing Zunera Design
  Foundation, application shell, navigation rules, and shared component
  guidelines.
- **FR-040**: Transaction experiences MUST remain readable and usable in Light,
  Dark, and System themes; across supported screen sizes; and with keyboard-only
  navigation, assistive technology, and high zoom.
- **FR-041**: The system MUST NOT lose or reinterpret historical transaction
  information when accounts or categories are archived or otherwise changed
  later, and MUST preserve enough information for financial reporting to remain
  consistent over time.
- **FR-042**: The system MUST let a user view the transactions they removed,
  clearly marked as removed, and MUST allow the user to restore one. A restored
  transaction MUST become pending or effective under the same state rules as a
  new transaction, and MUST affect balances only when effective. The interface
  MUST use the terms Remove, Removed, and Restore for this lifecycle.
- **FR-043**: A removed transaction MUST NOT be counted in account balances,
  totals, or filter results presented as current financial activity.
- **FR-044**: A transaction whose account or category has since been archived
  MUST remain fully editable, including its description, note,
  amount, type, account, category, date, and status, as long as that archived
  account or category keeps its existing association. Archived accounts and
  categories MUST be shown as read-only labels on such transactions, MUST NOT be
  selectable when the user chooses a different account or category, and MUST NOT
  become available to any other transaction. Moving a transaction away from an
  archived account or category requires selecting an active account and a
  matching active category.
- **FR-045**: Transaction history MUST load newest first in progressive batches
  of at most 50 entries, MUST show how many transactions match the current view,
  and MUST make every transaction reachable through progressive loading or
  search and filters. The system MUST support at least 5,000 transactions for a
  single user without losing access to earlier entries.
- **FR-046**: When an effective transaction's date is edited into the future,
  the system MUST preserve its effective state and its account-balance effect,
  MUST NOT change the state silently, and MUST show the user a clear notice that
  the transaction remains effective and still affects the balance, together
  with the option to set it to pending. Editing a pending transaction's date,
  including into the future, MUST leave it pending. This rule extends FR-020 to
  edits.
- **FR-047**: While a transaction is removed, the system MUST reject edits to it
  with a clear state conflict explaining that it must be restored first. Once
  restored, every editable field MUST be changeable again under the normal
  rules.
- **FR-048**: Archiving an account or category MUST NOT be blocked by, and MUST
  NOT change, the state or balance effect of transactions associated with it.
  Pending transactions MUST stay pending and remain visible, understandable, and
  editable while keeping that archived association. This feature MUST NOT add
  new blocking rules to the account or category archive behavior.
- **FR-049**: Every create, update, remove, and restore request MUST carry an
  action-specific idempotency key. Retrying the same request with that key MUST
  replay its original result without a second balance effect; reusing the key
  for a different request MUST be rejected. This MUST NOT prevent a user from
  intentionally recording two identical-looking transactions with different
  keys.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Every transaction action MUST require authentication and MUST
  default to denial when signed-in state or ownership of the transaction,
  account, or category cannot be confirmed.
- **SQR-002**: Every user-provided transaction value and requested action MUST
  be validated before it can create, change, remove, or affect the balance of a
  financial account.
- **SQR-003**: Unauthorized transaction access feedback MUST not reveal another
  user's transaction descriptions, amounts, dates, notes, account or category
  associations, existence, or balance effects.
- **SQR-004**: Automated coverage MUST prove ownership boundaries, validation
  rules, type-and-category compatibility, active-versus-archived selection
  rules, status transition rules, balance consistency across create, edit,
  status change, account change, type change, and removal, removal semantics,
  filtering and search behavior, historical-association preservation, and each
  primary user journey.
- **SQR-005**: Transaction data, balance effects, and transaction-to-account and
  transaction-to-category associations MUST be handled only within
  authenticated financial-management experiences, and MUST NOT be used to infer
  another user's financial position.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define and protect transaction behavior,
   ownership and authorization, validation rules, type-and-category
   compatibility, account and category selection rules, financial status rules,
   balance consistency, removal semantics, filtering and search behavior,
   historical association preservation, and automated verification before
   frontend delivery begins.
2. **Frontend** (`../zunera-frontend`): Deliver transaction recording, history,
   detail, editing, removal, filtering, and search journeys with feedback,
   responsive and accessible theme-aware presentation, and automated
   verification that consume the confirmed backend behavior.

### Key Entities *(include if feature involves data)*

- **Transaction**: A recorded financial movement affecting exactly one of the
  authenticated user's financial accounts. It has a type, a financial status, a
  category association, a description, a positive monetary amount, a
  transaction date, and an optional note, and it belongs to exactly one user
  through their financial context.
- **Transaction Type**: The financial classification of a movement. This
  feature supports income (money entering an account) and expense (money leaving
  an account) only, and must remain extensible to additional movement types
  without changing the meaning of existing transactions.
- **Transaction Financial Status**: Whether a financial movement has effectively
  occurred. Exactly two states exist: pending (planned or not yet settled) and
  effective (the movement has occurred). Only effective transactions affect
  account balances.
- **Transaction Removal Lifecycle**: Whether a transaction is active in the
  user's history or has been removed. A removed transaction keeps its data,
  stays out of balances and normal history, and can be restored by its owner
  without losing its historical information.
- **Monetary Amount**: The positive value of a movement, expressed in Brazilian
  currency conventions without loss of precision. The transaction type, not the
  sign of the amount, determines the direction of the financial effect.
- **Transaction Date**: The date on which the financial event occurred or is
  expected to occur, used for chronological presentation, filtering, and ranges.
- **Account Association**: The retained link between a transaction and the
  financial account it affects, owned by the same user. It remains meaningful
  when the account is later archived.
- **Category Association**: The retained link between a transaction and a
  category whose financial classification matches the transaction type,
  available to the same user. It remains meaningful when the category is later
  archived.
- **Balance Effect**: The derived change a financially effective transaction
  contributes to its account balance — an increase for income and a decrease for
  expense — always consistent with the account's original balance.
- **Transaction History View**: The user-facing chronological presentation of
  transactions with the descriptive and financial information needed to
  interpret them.
- **Transaction Filter and Search Criteria**: The optional date, date range,
  type, account, category, and status selections, plus the description-and-note
  text criteria, that a user applies to narrow or locate their transactions.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of acceptance-test users can record a valid income or
  expense transaction, including choosing account and category, in under 1
  minute without assistance.
- **SC-002**: In balance-consistency validation, 100% of create, edit, type
  change, account change, status change, and removal scenarios leave every
  affected account balance equal to its original balance plus its financially
  effective movements.
- **SC-003**: In authorization and privacy validation, 100% of attempts to view,
  edit, remove, filter by, or search another user's transactions, accounts, or
  custom categories are denied without revealing that data.
- **SC-004**: At least 90% of acceptance-test users can locate a specific
  transaction using filters or description search in under 30 seconds once their
  history contains at least 50 transactions.
- **SC-005**: In validation coverage, 100% of invalid amounts, dates, account or
  category associations, type-and-category combinations, and status transitions
  are rejected with feedback that identifies the field to correct.
- **SC-006**: In historical-integrity validation, 100% of transactions linked to
  subsequently archived accounts or categories remain accessible, correctly
  categorized, and correctly reflected in derived balances.
- **SC-007**: In responsive, theme, and accessibility validation, all primary
  transaction journeys remain usable at supported screen sizes, in Light, Dark,
  and System themes, and with keyboard-only navigation; income and expense remain
  distinguishable without color alone.
- **SC-008**: A first-time user can identify, for any transaction in their
  history, its description, amount, type, date, account, category, and status
  within 10 seconds without opening any other screen.
- **SC-009**: In removal and restoration validation, 100% of removed transactions
  stop affecting balances and normal history yet remain restorable with their
  original information, and 100% of restored effective transactions apply their
  balance effect exactly once.
- **SC-010**: In scale validation, users with 5,000 transactions see the first
  batch of their history within 2 seconds of opening it, and 100% of their
  transactions remain reachable through progressive loading, search, and
  filters.

## Assumptions

- Users already have an authenticated Zunera account and at least one financial
  account available before they can record transactions.
- Financial accounts, their initial balances, and their lifecycle behavior are
  defined by the Financial Accounts feature; this feature consumes that behavior
  and does not redefine it.
- Categories, their income and expense classifications, their ownership, and
  their archive/restore lifecycle are defined by the Category Management
  feature; this feature consumes that behavior and does not redefine it.
- Accounts and categories archived elsewhere remain historically valid for
  existing transactions but are not normally offered for new or changed
  associations. A transaction that keeps an archived account or category
  association stays fully editable in every field.
- Transactions are individual movements only. Recurring transactions,
  installment purchases, credit card invoices, budgets, financial goals, bank
  synchronization, automatic imports, advanced reports, and transfers between
  accounts are out of scope for this feature and belong to dedicated future
  features.
- A single currency (Brazilian Real) is used. Multi-currency transactions,
  currency conversion, and exchange-rate handling are out of scope.
- Transactions have two states only: pending and effective. Newly recorded
  transactions default to effective, and future-dated transactions default to
  pending and stay out of balances until the user marks them effective.
- Removal is a lifecycle change rather than permanent deletion, so that
  balances, history, and previously reported information stay internally
  consistent and a mistaken removal can be corrected. The interface uses
  Remove, Removed, and Restore as its user-facing terms.
- Users record financial movements manually in this feature. Imported or
  automatically created transactions are out of scope.
- A transaction amount is stored and displayed with exactly two decimal places,
  from R$ 0.01 to R$ 999,999,999.99. Larger, smaller, zero, or negative values
  are rejected.
- Transaction dates from 1900-01-01 through 2100-12-31 are supported; dates
  outside that range are rejected so that implausible history cannot be
  recorded.
- Duplicate-looking transactions are legitimate user choices; the system
  prevents only the duplicate financial effect caused by repeating the same
  action, not two intentionally similar transactions.
- Transaction search matches the description and note as literal text, ignoring
  letter case and accent differences. Category, account, type, and status are
  not searchable as text; they are selected as filters.
- A user's history may grow to thousands of transactions. History is presented
  newest first in progressive batches of at most 50 entries, and the feature is
  expected to handle at least 5,000 transactions per user.
- Attachments, receipt images, tags, split transactions, and shared or
  household transactions are out of scope for this feature.
