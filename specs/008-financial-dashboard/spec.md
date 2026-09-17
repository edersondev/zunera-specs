# Feature Specification: Financial Dashboard

**Feature Branch**: `008-financial-dashboard`  
**Backend Branch**: `008-financial-dashboard` (`../zunera-backend`)  
**Frontend Branch**: `008-financial-dashboard` (`../zunera-frontend`)  
**Created**: 2026-09-17  
**Status**: Draft  
**Input**: User description: "Create the Financial Dashboard feature for Zunera."

## Clarifications

### Session 2026-09-17

- Q: What reporting period opens initially? → A: Current month, from its first
  day through current business date.
- Q: How should archived accounts appear in dashboard account overview? → A:
  Archived accounts appear only where needed to explain valid historical period
  information; they never appear in current account overview.
- Q: What upcoming activity horizon should initial dashboard use? → A: Next 30
  calendar days.
- Q: Which existing data supplies upcoming activity? → A: Valid future pending
  transactions and eligible future recurring occurrences.
- Q: Should income distribution by category appear initially? → A: No; initial
  dashboard includes income total and evolution, but no income distribution.
- Q: How should financial-evolution intervals adapt to reporting-period length?
  → A: Daily through 31 days, weekly through 93 days, monthly beyond; partial
  boundary intervals are clearly identified and not compared as complete.
- Q: How should past or current pending transactions appear in recent activity?
  → A: Include them with a clear Pending label.
- Q: What user-visible performance target applies to dashboard loading and period
  changes? → A: Usable results within 2 seconds for up to 10,000 financial
  movements.
- Q: How much recent activity should dashboard show? → A: Ten newest items with
  a link to full Financial History.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - See Current Position and Period Result (Priority: P1)

As an authenticated user, I want to see my current total balance and realized
income, expenses, and financial result for a reporting period, so that I can
understand my current position and recent financial performance quickly.

**Why this priority**: This is the dashboard's essential answer to whether the
user has money now and whether realized activity in the selected period gained
or spent money.

**Independent Test**: A signed-in user with active accounts and effective income,
expense, and transfer records can open the dashboard, select a period, and
verify current total balance plus accurate realized income, expenses, and result.

**Acceptance Scenarios**:

1. **Given** a user owns active accounts and effective income and expense
   transactions in the selected period, **When** they view the dashboard,
   **Then** they see current total balance, realized income, realized expenses,
   and result calculated as income minus expenses.
2. **Given** a user owns an effective transfer between their accounts, **When**
   they view any summary containing that movement, **Then** the transfer changes
   neither income nor expenses nor combined current balance.
3. **Given** a user changes the reporting period, **When** refreshed information
   is presented, **Then** every period-dependent summary, distribution, and
   evolution value uses the same selected dates while current total balance is
   clearly labelled as current.
4. **Given** a user has only pending records, planned recurrences, or other
   non-effective records in a selected period, **When** they view realized
   summary values, **Then** those records do not contribute to realized income,
   expenses, result, or current balance.

---

### User Story 2 - Understand Spending and Change Over Time (Priority: P2)

As an authenticated user, I want to understand expense distribution and the
evolution of my realized finances, so that I can identify where money went and
how my financial result changed over time.

**Why this priority**: Breakdown and trend context turns totals into useful
financial understanding without redefining transaction meaning.

**Independent Test**: A user with effective expenses across categories and dates
can select a period and identify each category's total, relative contribution,
largest spending categories, and income, expense, and result values over time.

**Acceptance Scenarios**:

1. **Given** a selected period contains realized expenses in multiple categories,
   **When** the user views expense distribution, **Then** every displayed total
   and relative contribution derives only from those realized expenses and the
   largest contributors are identifiable without color alone.
2. **Given** a category used by historical effective expenses is archived,
   **When** the user views a period containing those expenses, **Then** its
   historical contribution remains understandable and is identified as archived
   where relevant.
3. **Given** a selected period spans multiple comparable time intervals,
   **When** the user views financial evolution, **Then** income, expenses, and
   result for each interval follow the same realized-state rules and interval
   scope is clear.

---

### User Story 3 - Review Accounts and Recent Activity (Priority: P2)

As an authenticated user, I want to see where my current balance is held and
what recently happened, so that I can reconcile the dashboard with my accounts
and financial history.

**Why this priority**: Account allocation and recent movements help users trust
and act on the high-level summary.

**Independent Test**: A user with active accounts and mixed recent income,
expenses, and transfers can identify each account's balance and distinguish
recent movement type, amount, date, account, and applicable category.

**Acceptance Scenarios**:

1. **Given** a user owns active accounts, **When** they view account overview,
   **Then** each relevant active account shows identity, type, current balance,
   and its contribution to the displayed current total balance.
2. **Given** a user has recent income, expense, transfer, or past/current pending
   activity, **When** they view recent activity, **Then** each movement shows
   description, amount, type, date, account, and category when applicable, with
   type and pending state understandable without color alone, and no more than
   ten newest items are shown with access to full Financial History.
3. **Given** a recurring occurrence has created a financial transaction,
   **When** it appears in recent activity, **Then** it is shown as that resulting
   transaction rather than as a separate duplicate event.

---

### User Story 4 - Distinguish Expected Activity (Priority: P3)

As an authenticated user, I want to see upcoming expected movements separately
from realized activity, so that I can prepare for near-future income and
expenses without mistaking them for money already received or spent.

**Why this priority**: Expected activity is useful context but must never make
the current position or realized results misleading.

**Independent Test**: A user with valid future pending activity or eligible
recurring activity can see its date, type, amount, and context clearly marked
expected; a user without it sees an explanatory empty state.

**Acceptance Scenarios**:

1. **Given** a user has valid future pending transactions or eligible future
   recurring activity within the upcoming horizon, **When** they view the
   dashboard, **Then** it is marked expected and is excluded from current
   balance and all realized totals.
2. **Given** an expected occurrence later becomes an effective transaction,
   **When** dashboard information is refreshed, **Then** it no longer appears
   only as expected and contributes to realized information according to its
   transaction state and date.

### Edge Cases

- User has no financial accounts: current total is zero, account overview explains
  absence, and provides a path to create an account.
- User has accounts but no transactions, income, expenses, transfers, categories,
  recurring rules, or upcoming activity: each affected section explains absence
  without presenting invented totals or events.
- A selected period contains no realized income or no realized expense: relevant
  value is zero and distribution/evolution states remain clear and accessible.
- A user owns only pending, removed, or otherwise non-effective financial
  records: they remain excluded from realized amounts and current balances. An
  effective transaction whose date is edited into the future keeps its effective
  state, its balance effect, and its reported period according to its stored
  transaction date, as established by Transactions rules.
- Effective income or expense is corrected, removed, restored, or re-dated:
  affected dashboard information reflects the valid resulting financial state.
- An account is archived: it leaves the current account overview and current
  total according to account rules, while valid historical activity remains
  interpretable in period information.
- A category is archived after prior activity: the category remains understandable
  in applicable historical distributions.
- A recurrence is edited, paused, ended, or has a linked account/category
  archived: historical transaction occurrences preserve their established
  meaning; only eligible expected activity is shown.
- Transfer source and destination are both owned but one is archived after an
  effective transfer: historical transfer remains identifiable without changing
  its established combined-balance effect.
- A user attempts to access another user's dashboard information or a stale
  section response: no other user's account, transaction, transfer, category,
  recurrence, aggregate, or existence information is exposed.
- One section is unavailable while others remain reliable: available sections
  remain usable and the unavailable section gives clear retry guidance.
- Theme, narrow viewport, keyboard-only navigation, assistive technology, high
  zoom, and non-color perception do not prevent understanding dashboard content.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide dashboard information only to authenticated
  users and derive it exclusively from financial data owned by that user.
- **FR-002**: Dashboard MUST not create, store, or maintain independent financial
  balances, totals, movement effects, or lifecycle rules; all displayed values
  MUST derive from existing financial account, transaction, transfer, category,
  and recurring-transaction information and its established rules.
- **FR-003**: Dashboard MUST show current total balance as combined current
  balance of user's active financial accounts only. Archived accounts MUST not
  contribute to this current total or normally appear in current account overview.
- **FR-004**: Dashboard MUST provide current month, previous month, and custom
  inclusive date-range reporting choices.
- **FR-005**: Dashboard MUST default selected reporting period according to the
  user's established financial business date context to current month, from its
  first day through current business date, and clearly label its start and end
  dates.
- **FR-006**: Changing reporting period MUST update all period-dependent
  dashboard sections consistently. Current total balance MUST remain labelled as
  current and MUST not be implied to be a historical period-end balance.
- **FR-007**: Dashboard MUST show total realized income for selected period from
  owned income transactions that are financially effective and dated within that
  period. Transfers, pending records, planned recurrences, and other non-effective
  records MUST be excluded.
- **FR-008**: Dashboard MUST show total realized expenses for selected period
  from owned expense transactions that are financially effective and dated within
  that period. Transfers, pending records, planned recurrences, and other
  non-effective records MUST be excluded.
- **FR-009**: Dashboard MUST show financial result for selected period as
  realized income minus realized expenses, and communicate positive, zero, and
  negative results in text or symbols as well as semantic color.
- **FR-010**: Dashboard MUST provide selected-period expense distribution by
  category using only realized expenses. It MUST show category spending total
  and relative contribution, identify largest contributors, and remain
  understandable without color alone.
- **FR-011**: Dashboard MUST provide financial evolution across clearly labelled
  comparable intervals within selected period, showing realized income, expenses,
  and financial result for each interval. It MUST use daily intervals through 31
  selected days, weekly intervals through 93 selected days, and monthly intervals
  for longer periods. A partial boundary interval MUST be clearly identified and
  MUST NOT be compared visually as a complete interval.
- **FR-012**: Dashboard MUST provide current overview of active owned accounts
  with account identity, type, current balance, and balance allocation. It MUST
  exclude archived accounts by default while retaining valid archived-account
  activity in historical period sections. It MUST NOT offer archived accounts in
  current account overview. An account allocation percentage is the signed
  account balance divided by the combined active balance when that total is
  non-zero; it MAY be negative or greater than 100%. When the combined active
  balance is zero, allocation percentage MUST be unavailable rather than an
  invented value.
- **FR-013**: Dashboard MUST provide recent owned financial activity, including
  income, expenses, transfers, and past/current pending transactions, with
  description, amount, movement type, date, account, and applicable category.
  Pending transactions MUST have a clear Pending label. Recurring activity MUST
  be represented by resulting transactions, not duplicate recurrence events. It
  MUST show the ten newest items when available (never more than ten) and
  provide a link to full Financial History.
- **FR-014**: Dashboard MUST identify income, expenses, transfers, effective,
  pending, and expected information with text, labels, icons, or equivalent
  non-color cues; color alone MUST never convey those distinctions.
- **FR-015**: Dashboard MUST show upcoming owned financial activity only as
  planned or expected and never include it in current balances, realized income,
  realized expenses, realized result, expense distribution, or realized evolution.
- **FR-016**: Dashboard MUST show upcoming activity for next 30 calendar days,
  inclusive of current business date, and label that horizon clearly.
- **FR-017**: Dashboard MUST surface valid owned future pending transactions and
  eligible future recurring occurrences when their established source rules make
  them valid. A recurrence rule alone must not be represented as realized
  activity.
- **FR-018**: Dashboard MUST provide section-specific loading, empty, and error
  feedback. Failure of one section MUST not prevent reliable independent
  dashboard information from remaining available.
- **FR-019**: Empty states MUST explain absence and, where useful, offer next
  action for no accounts, no transactions, no income, no expenses, no recurring
  transactions, and no upcoming activity.
- **FR-020**: Dashboard historical information MUST preserve valid meanings when
  accounts or categories archive; transactions change lifecycle, are edited,
  removed, or restored; or recurring rules change, pause, or end. It MUST reflect
  subsequent valid corrections rather than retain stale independent values.
- **FR-021**: Dashboard MUST follow Zunera Design Foundation, application shell,
  navigation, and shared interface guidance, including financial semantic colors,
  accessible chart treatment, Light/Dark/System themes, responsive priority for
  key summary information, keyboard operation, assistive technology, focus
  visibility, contrast, and supported zoom levels.
- **FR-022**: Dashboard MUST not include budget management, financial goals,
  advanced reports, cash-flow forecasting, investment performance, credit-card
  invoice analysis, bank synchronization, automatic import, recommendations,
  AI-generated insights, notifications, or income distribution by category.
- **FR-023**: Dashboard MUST remain able to accept future summaries from
  dedicated financial features without changing its core definitions of current
  balance, realized income, realized expenses, realized result, transfer effect,
  or ownership.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Every dashboard request and derived result MUST require confirmed
  authentication and user ownership. Missing or unconfirmed ownership MUST deny
  access without exposing another user's financial information or existence.
- **SQR-002**: Any user-provided period choice or dashboard filter MUST be
  validated before it influences displayed data; invalid values MUST receive
  privacy-safe correction feedback and must not produce misleading aggregates.
- **SQR-003**: Automated coverage MUST prove ownership isolation, effective versus
  pending/planned treatment, transfer exclusion, current balance, account/category
  archival behavior, calculation corrections, period consistency, recent and
  upcoming activity distinctions, empty/error/loading states, accessibility
  cues, and critical user journeys.
- **SQR-004**: Dashboard information is personal financial information. No
  secrets, uploads, external financial-provider access, or cross-user aggregates
  are in scope.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define dashboard business behavior,
   ownership, validated period handling, established-state derivation,
   historical integrity, section-specific results/failures, financial consistency,
   contract, and automated tests before frontend delivery.
2. **Frontend** (`../zunera-frontend`): Deliver accessible responsive dashboard
   hierarchy, period selection, summaries, account and activity views,
   visualizations, realized/expected distinction, empty/loading/error feedback,
   theme support, and automated tests consuming confirmed backend contract.

### Key Entities *(include if feature involves data)*

- **Dashboard View**: User-specific derived presentation of existing financial
  state. It holds no independent financial state.
- **Reporting Period**: Selected inclusive calendar-date scope applied
  consistently to realized summary, distribution, and evolution information.
- **Current Total Balance**: Sum of current balances of active owned accounts,
  independent of reporting period and unchanged in aggregate by owned transfers.
- **Realized Financial Summary**: Selected-period income, expenses, and result,
  based exclusively on financially effective income/expense transactions.
- **Expense Distribution**: Selected-period realized expense totals and shares by
  category, including historical understanding for archived categories.
- **Financial Evolution Interval**: Clearly labelled daily, weekly, or monthly
  part of reporting period with realized income, expenses, and result; partial
  boundary interval is distinguished from a complete interval.
- **Recent Activity Item**: Existing user-owned income, expense, or transfer,
  including recurring occurrence represented through resulting transaction.
- **Upcoming Activity Item**: Existing valid future planned or expected movement,
  explicitly separate from realized financial state.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of acceptance-test users with representative account
  and transaction data identify current total balance, period income, expenses,
  and result within 30 seconds without assistance.
- **SC-002**: In calculation acceptance tests covering effective income, expense,
  pending activity, planned recurrences, and transfers, 100% of displayed
  summary, distribution, and evolution values match established underlying
  financial state.
- **SC-003**: At least 90% of acceptance-test users correctly distinguish a
  realized movement from an expected movement and a transfer from income or
  expense without relying on color.
- **SC-004**: At least 90% of acceptance-test users can change between current
  month, previous month, and a custom range and confirm all period-dependent
  information changed consistently on first attempt.
- **SC-005**: In accessibility acceptance tests across supported themes and
  viewport sizes, 100% of primary dashboard information and movement types
  remain understandable by keyboard-only and non-color-dependent review.
- **SC-006**: Users with up to 10,000 owned financial movements see usable
  dashboard information after opening it or changing reporting period within 2
  seconds in acceptance performance validation.

## Assumptions

- Existing Financial Accounts, Categories, Transactions, Transfers, and Recurring
  Transactions specifications remain source of truth for ownership, lifecycle,
  dates, balances, classification, and financial effectiveness.
- Current month, previous month, and custom inclusive date range are minimum
  reporting choices; current month opens by default.
- Current total balance is sum of active accounts only, as established by
  Financial Accounts; archived-account financial movements remain available in
  valid historical period analysis.
- Current balance is intentionally distinct from selected-period reporting and
  is labelled as current.
- Realized means financially effective under Transactions rules. Pending and
  expected recurring information is never silently promoted to realized state;
  an effective transaction re-dated into the future remains effective and is
  reported in the period containing its stored date.
- Monetary values retain existing Brazilian-real cent precision and financial
  business-date rules.
- Planned/expected information can be absent without reducing availability of
  realized dashboard sections.
