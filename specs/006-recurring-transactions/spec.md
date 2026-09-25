# Feature Specification: Recurring Transactions

**Feature Branch**: `006-recurring-transactions`  
**Backend Branch**: `006-recurring-transactions` (`../zunera-backend`)  
**Frontend Branch**: `006-recurring-transactions` (`../zunera-frontend`)  
**Created**: 2026-09-14  
**Status**: Draft  
**Input**: User description: "Create the Recurring Transactions feature for
Zunera."

## Clarifications

### Session 2026-09-14

- Q: After processing downtime, how are eligible missed due dates handled? → A:
  Create one pending occurrence for every eligible missed due date.
- Q: What happens when a linked account or category is archived? → A:
  Automatically pause the recurrence.
- Q: What happens after a recurrence's inclusive end date passes? → A:
  Automatically mark the recurrence ended.
- Q: How are eligible dates that passed while a rule was paused treated when it
  resumes? → A: Skip them permanently; dates inside the paused window are never
  evaluated or created after resume.
- Q: Which next expected occurrence is shown for paused, ended, or
  end-date-exhausted rules? → A: None; a next expected occurrence exists only
  while a rule is active and a future eligible date remains.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create a Recurring Income or Expense (Priority: P1)

As an authenticated user, I want to define a repeating income or expense with
its account, category, amount, schedule, and dates, so that I do not have to
recreate predictable financial events one by one.

**Why this priority**: Recurrence definition is foundation for every later
occurrence and management action.

**Independent Test**: A signed-in user with eligible account and category saves
a weekly, monthly, or yearly recurrence and can see its complete definition and
next expected occurrence.

**Acceptance Scenarios**:

1. **Given** a user has an active account and active income category, **When**
   they create a valid income recurrence, **Then** it belongs only to them,
   appears active, and shows its next expected occurrence.
2. **Given** a user has an active account and active expense category, **When**
   they create a valid expense recurrence, **Then** its category matches its
   classification and success feedback is shown.
3. **Given** a user submits an invalid amount, date range, account, category,
   or category classification, **When** they save, **Then** no rule is created
   and clear field-level feedback explains correction.
4. **Given** a user creates a rule with a future start date, **When** it is
   saved, **Then** no occurrence exists before the start and its schedule is
   visible.

---

### User Story 2 - See Scheduled Occurrences in History (Priority: P1)

As an authenticated user, I want due recurring events to appear as ordinary
financial transactions with clear origin information, so that recurring and
one-time events form one understandable financial history.

**Why this priority**: A recurrence is valuable only when its occurrences are
reliable financial movements governed by transaction rules.

**Independent Test**: A user with a due rule finds its occurrence in ordinary
history, opens it, recognizes its source rule, and confirms its balance effect
follows the established transaction lifecycle.

**Acceptance Scenarios**:

1. **Given** an active rule reaches a scheduled date, **When** its occurrence
   is processed, **Then** exactly one pending ordinary transaction is
   identifiable as originating from that rule and date and awaits the owner's
   ordinary transaction confirmation to become effective.
2. **Given** a generated occurrence appears in history, **When** the user opens
   it, **Then** type, amount, account, category, description, date, status, and
   recurrence origin are understandable.
3. **Given** only a recurrence definition exists, **When** account balances are
   viewed, **Then** that definition has no direct balance effect.
4. **Given** processing is retried or overlaps, **When** history and balances
   are checked, **Then** no duplicate transaction or balance effect exists.

---

### User Story 3 - Manage Future Recurrence Rules (Priority: P2)

As an authenticated user, I want to edit, pause, resume, and end my recurring
rules, so that plans change without rewriting financial history.

**Why this priority**: Repeating commitments change; lifecycle controls protect
past financial events while keeping rules useful.

**Independent Test**: A user edits an active rule, pauses it, resumes it, and
ends it, while verifying historical occurrences stay unchanged and paused dates
are not backfilled.

**Acceptance Scenarios**:

1. **Given** a user owns an active rule, **When** they make a valid edit,
   **Then** it applies only to future ungenerated dates and success feedback is
   shown.
2. **Given** a user pauses a rule, **When** dates pass during the pause,
   **Then** history stays unchanged and no missed occurrence is auto-created.
3. **Given** a user resumes a paused rule, **When** it resumes, **Then** it
   follows its next future scheduled date without automatic backfill.
4. **Given** a user ends an active or paused rule, **When** termination succeeds,
   **Then** no later occurrence is created and historical occurrences remain.
5. **Given** an active rule reaches the end of its inclusive end date, **When**
   its final eligible schedule date has passed, **Then** it automatically becomes
   ended and no later occurrence is created.

---

### User Story 4 - Correct One Occurrence (Priority: P2)

As an authenticated user, I want to correct one exceptional occurrence without
changing the recurring pattern, so that a variation does not affect later dates.

**Why this priority**: Exceptional bills and payments are common and must not
corrupt the normal pattern.

**Independent Test**: A user changes one generated occurrence and verifies the
source rule and later occurrences retain their scheduled values.

**Acceptance Scenarios**:

1. **Given** a R$ 100,00 monthly rule, **When** the user corrects April's
   generated transaction to R$ 115,00, **Then** April retains R$ 115,00 while
   the rule and May remain R$ 100,00 unless separately changed.
2. **Given** a user can edit a source rule or a generated transaction, **When**
   either action is selected, **Then** the interface makes its scope clear
   before saving.

---

### User Story 5 - Find and Understand Recurrences (Priority: P3)

As an authenticated user, I want to filter recurring transactions and see their
status, schedule, account, and next expected date, so that I can organize
ongoing income and commitments.

**Why this priority**: Organization matters after recurring activity is
reliable.

**Independent Test**: A user with active, paused, and ended rules can combine
filters and identify the next expected occurrence of every matching rule.

**Acceptance Scenarios**:

1. **Given** rules with different classifications, accounts, categories,
   frequencies, and states, **When** filters are combined, **Then** only rules
   matching every criterion are listed and criteria can be cleared.
2. **Given** a mixed rule list, **When** it is viewed, **Then** every entry
   communicates type, amount, frequency, account, category, state, and next
   expected occurrence when one remains.
3. **Given** recurring rules on desktop or mobile, **When** the owner scans or
   expands the list, **Then** each compact item shows description, destination,
   category, signed amount, frequency, next occurrence, and lifecycle state;
   expansion reveals existing rule details and a route to occurrence history.
4. **Given** applied filters, **When** the filter panel is collapsed, **Then**
   its header shows the number of active criteria while the recurrence list
   retains priority on the page.

### Edge Cases

- User has no active account or matching active category.
- Start date is today, future, or past. Eligibility begins on the later of the
  start date and the rule's creation date, so a past start never auto-creates
  historical transactions and elapsed dates are skipped.
- Monthly 29th, 30th, or 31st uses the month’s final calendar day if its ordinal
  date is absent, then returns to its ordinal day when available. Yearly 29
  February uses 28 February in non-leap years and 29 February in leap years.
- End date precedes start, occurs before a future due date, or is reached while
  rule is active or paused. After its inclusive end date passes, rule becomes
  ended automatically.
- Rule is paused, resumed, or ended repeatedly; user tries to resume or edit an
  ended rule.
- Rule is paused. No next expected occurrence is shown, nothing is generated
  while paused, and dates that pass during the pause stay skipped after resume.
- Associated account or category is later archived. Rule and history remain;
  rule automatically pauses and new occurrences stop until owner provides
  eligible active association and resumes it.
- Generated occurrence is individually edited, removed, restored, or changes
  pending/effective state under ordinary transaction rules.
- Rule is edited on a due date, processing retries after partial completion, or
  processing overlaps for same rule/date.
- Rule start date is changed earlier or later, or end date is extended, cleared,
  or shortened. Only ungenerated future dates change; generated occurrences keep
  their snapshots and never become eligible again.
- Processing resumes after downtime. Every eligible due date missed while the
  rule remained active is created once as a pending occurrence, never as an
  automatic balance effect.
- User attempts to access another user’s rules, associations, or occurrences.
- Inputs are blank, malformed, overlong, out of range, or use invalid Brazilian
  currency formatting.
- Journeys run in Light, Dark, System theme; narrow screens; keyboard-only,
  assistive technology, or high zoom.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST let authenticated users create, list, filter, view,
  update, pause, resume, and end only recurring transactions they own.
- **FR-002**: A recurrence MUST belong to exactly one user and represent one
  repeating income or expense rule. It is not an ordinary transaction and MUST
  NOT directly affect account balance.
- **FR-003**: Each rule MUST record classification, positive amount, one owned
  financial account, one matching available category, description, start date,
  frequency, optional end date, and optional note.
- **FR-004**: System MUST support weekly, monthly, and yearly frequencies.
  Daily, biweekly, quarterly, custom intervals, transfers, card installments,
  loans, invoices, sync, imports, notifications, budgets, and goals are out of
  scope.
- **FR-005**: Income rule MUST use income category; expense rule MUST use
  expense category. System MUST reject mismatches, unavailable categories, and
  another user’s personal categories without changing rule.
- **FR-006**: Creation or reassignment MUST use active owned account. System
  MUST reject archived or another user’s account as new association.
- **FR-007**: Creation or reassignment MUST use active matching category
  available to user. Archived categories MUST NOT be selectable as new
  association.
- **FR-008**: If associated account or category later archives, rule MUST remain
  understandable with archived label and historical occurrences unchanged. Rule
  MUST automatically enter paused state, stop creating new occurrences, show
  association-unavailable reason, and allow owner to reassign active association
  before resuming.
- **FR-009**: Amount MUST be positive Brazilian-real value from R$ 0,01 through
  R$ 999.999.999,99 with at most two decimals. Classification, not sign,
  determines direction. Zero, negative, malformed, over-precision, and out of
  range amounts MUST be rejected without cent precision loss.
- **FR-010**: Description MUST contain 1–200 visible characters after trimming
  and non-whitespace content. Optional note MUST contain at most 1.000 visible
  characters after trimming. Invalid text MUST receive correction feedback.
- **FR-011**: Start date MUST be 1900-01-01 through 2100-12-31. Optional end
  date MUST be in same range and on or after start. Missing, malformed, or
  invalid dates MUST be rejected.
- **FR-012**: Future start creates no earlier occurrence. Current-date start is
  eligible that day. Past start creates no automatic history; elapsed scheduled
  dates are skipped and next unelapsed scheduled date is first eligible date.
  Schedule eligibility begins on the later of the start date and the business
  date the rule was created, so dates before that eligibility start are never
  evaluated or created.
- **FR-013**: Weekly schedule uses start-date weekday. Monthly schedule uses
  start-date ordinal, or final day when missing. Yearly schedule uses start
  month/day, with 28 February for 29 February in non-leap years.
- **FR-014**: Only a scheduled date on/after the eligibility start defined by
  FR-012 and on/before optional end is eligible for processing; FR-015 governs
  exactly-once creation for an eligible active date. No occurrence may occur
  after end; ending prevents future occurrences without altering history. A rule
  with end date MUST automatically become ended after its inclusive end date
  passes.
- **FR-015**: When an active recurrence reaches its due date, the system MUST
  create exactly one pending generated transaction for that date. It MUST remain
  pending, with no balance effect, until its owner marks it effective through
  ordinary transaction management. When processing resumes after downtime, it
  MUST create one pending occurrence for every eligible due date missed while the
  rule was active, from its eligibility start through the current business date.
  Dates that passed while the rule was paused are not missed due dates and MUST
  NOT be created. The system MUST NOT create occurrences before their due dates
  or automatically make a generated occurrence effective.
- **FR-016**: Generated occurrence MUST be ordinary transaction governed by
  established transaction type, status, balance, removal, restoration,
  ownership, and historical-integrity rules. It MUST snapshot rule type, amount,
  account, category, description, note, and scheduled financial date.
- **FR-017**: Generated occurrence MUST visibly identify its originating rule
  without color-only meaning. User MUST recognize source from history and
  details and open occurrence from source rule. An occurrence removed under
  ordinary transaction rules MUST stay identifiable from its rule as removed, so
  a skipped schedule date remains explainable.
- **FR-018**: System MUST prevent more than one generated transaction for same
  rule and scheduled date, including retries, concurrent processing, or repeated
  requests; separate rules and intentional ordinary transactions remain allowed.
- **FR-019**: Rule lifecycle state MUST be active, paused, or ended. Active is
  schedule-eligible; paused retains definition but creates nothing; ended retains
  definition/history but never creates another occurrence. A rule automatically
  enters ended state after its inclusive end date passes.
- **FR-020**: Owner MUST be able to pause active rule and resume paused rule if
  dates/associations remain eligible. Dates missed while paused are skipped;
  resuming MUST NOT auto-create or financially effect them, and resumed
  processing MUST consider only dates on or after the resume date so dates
  inside the paused window are never evaluated again.
- **FR-021**: Owner MUST be able to end active or paused rule permanently. Ended
  rule cannot resume or edit; viewing retained detail/history stays allowed;
  unavailable lifecycle action MUST receive clear state feedback.
- **FR-022**: Valid active-rule edit applies only to scheduled dates not already
  represented by generated transaction. Past/generated occurrences preserve
  snapshot, status, and financial effect. Editing the start date re-anchors the
  schedule, including the weekday used by weekly rules, from the rule’s
  eligibility start onward; editing the end date may extend, shorten, or clear
  it. No edit may make a generated date eligible again or revive an ended rule.
- **FR-023**: User MUST correct generated occurrence through ordinary transaction
  management without changing source rule or other occurrences. Rule editing and
  one-occurrence editing MUST be clearly distinguished.
- **FR-024**: List MUST support combined filters: classification, account,
  category, frequency, lifecycle state. Each entry MUST show description, amount,
  classification, frequency, account, category, state, and next expected date
  when one remains. A next expected occurrence exists only while the rule is
  active and a future eligible date remains within its start, end, and calendar
  rules; paused and ended rules MUST show none. List MUST be ordered by next
  expected occurrence ascending, then rules without one in a stable order.
- **FR-025**: Rule/occurrence history MUST remain understandable if rule is
  edited, paused, ended, or associated account/category later archives. No rule
  action may silently rewrite historical transaction.
- **FR-026**: Create, update, pause, resume, end success MUST show confirmation.
  Invalid inputs/patterns/associations, mismatches, unauthorized requests, and
  state conflicts MUST provide privacy-safe actionable feedback without partial
  change.
- **FR-027**: Recurrence journeys MUST follow Zunera Design Foundation,
  application shell, navigation, shared component rules, be non-color-only, and
  work in Light, Dark, System themes, supported screen sizes, keyboard use,
  assistive technology, and high zoom.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Rule and occurrence action MUST require authentication; deny by
  default when rule, account, category, or occurrence ownership is unconfirmed.
- **SQR-002**: Submitted values, schedules, associations, state transitions,
  occurrence-generation decision, and individual-occurrence action MUST validate
  before changing financial data.
- **SQR-003**: Unauthorized feedback MUST not reveal another user’s rule,
  schedule, amount, description, note, account, category, occurrence, or effect.
- **SQR-004**: Automated coverage MUST prove ownership, validation, compatible
  classification, lifecycle, calendar, no historical backfill, uniqueness,
  balance consistency, historical integrity, filters, individual corrections,
  contract changes, and primary user journeys.
- **SQR-005**: Recurrence data, occurrences, and effects are personal financial
  information used only in authenticated financial-management experiences. No
  secret, upload, or external-provider behavior is in scope.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define and protect ownership, validation,
   calendar/lifecycle rules, generated-transaction relationship, duplicate
   prevention, balance consistency, historical integrity, and automated
   verification before frontend delivery.
2. **Frontend** (`../zunera-frontend`): Deliver create/list/detail/update/
   lifecycle/filter/origin/feedback journeys with responsive accessible
   theme-aware presentation and automated verification consuming confirmed
   backend behavior.

### Key Entities *(include if feature involves data)*

- **Recurring Transaction**: User-owned recurring income/expense definition;
  carries financial information and schedule, never direct balance effect.
- **Recurring Schedule**: Weekly, monthly, or yearly rule with start, optional
  inclusive end, and defined calendar adjustment.
- **Generated Occurrence**: One ordinary transaction for one rule and schedule
  date; preserves source snapshot, transaction lifecycle, and source link.
- **Recurrence Lifecycle State**: Active, paused, or ended; governs future
  eligibility without rewriting history.
- **Occurrence Source Link**: Visible retained rule/date relationship enabling
  duplicate prevention and historical interpretation.
- **Next Expected Occurrence**: Earliest eligible future schedule date under
  start/end, calendar, association, and lifecycle rules.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% acceptance-test users create valid weekly, monthly,
  or yearly income/expense rule in under 2 minutes without assistance.
- **SC-002**: At least 90% users identify rule type, amount, account, status,
  frequency, and next expected date in under 30 seconds.
- **SC-003**: In processing validation, 100% retry/concurrent attempts for one
  due schedule produce at most one transaction and one balance effect.
- **SC-004**: In integrity validation, 100% edits, pauses, resumes, ends, and
  later association archives preserve occurrence snapshots and prior effects.
- **SC-005**: At least 90% users pause, resume, or end a rule and accurately
  predict whether next schedule period will contain occurrence without help.
- **SC-006**: In authorization validation, 100% cross-user rule/occurrence
  access attempts are denied without personal financial disclosure.
- **SC-007**: All primary journeys remain usable in supported themes, screen
  sizes, keyboard-only navigation, and assistive technology validation.

## Assumptions

- Users already have authenticated Zunera account plus existing account,
  category, and transaction behavior.
- Personal income/expense recurrence only; shared/business, transfers, card,
  loans, invoices, sync/import, external cancellation, budgets/goals, reminders,
  and notifications are outside scope.
- Existing transaction rules remain authoritative: pending has no balance effect;
  effective does; removed stays retained outside normal history/balances; and
  individual occurrence corrections use same rules.
- Past starts and paused dates never auto-backfill. User records a missed past
  movement individually if needed.
- Rule edits affect only ungenerated dates. One generated occurrence is changed
  through ordinary transaction management.
- Archived association preserves history but blocks further scheduled occurrence
  until active replacement and resume.
- Due dates create one pending occurrence only on that date. Owners use ordinary
  transaction management to mark it effective; scheduled occurrences are never
  created in advance or made effective automatically.
