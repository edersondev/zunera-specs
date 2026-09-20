# Feature Specification: Credit Cards

**Feature Branch**: `010-credit-cards`  
**Backend Branch**: `010-credit-cards` (`../zunera-backend`)  
**Frontend Branch**: `010-credit-cards` (`../zunera-frontend`)  
**Created**: 2026-09-20  
**Status**: Ready for planning  
**Input**: User description: "Create the Credit Cards feature for Zunera."

## Clarifications

### Session 2026-09-20

- Q: Does a purchase made on closing date belong to statement closing that day,
  or next statement? → A: Closing date is included in current statement; cycle
  ends after that day.
- Q: When does a credit-card purchase consume category spending and monthly
  budget? → A: Each installment is recognized in its associated statement
  period. Statement payment is settlement, never a second expense.
- Q: Does initial release include traceable full/partial refunds and
  post-closing corrections? → A: Yes. They create traceable credit events and
  restate affected statements, obligations, recognized spending, and available
  credit without altering original history.
- Q: How is a statement due date determined from closing and due days? → A: It
  is first configured due day after closing: same month when later, otherwise
  next month.
- Q: How does a refund-created card credit affect available credit? → A: It is
  shown separately, adds usable credit, and may make available credit exceed
  card's stated limit.
- Q: How is available card credit applied against unpaid statements? → A: Apply
  it automatically to oldest unpaid statement; retain any remainder for next
  statement.
- Q: How should current cycle appear before any card activity? → A: Every active
  card shows an Open R$ 0,00 current statement until activity occurs.
- Q: How should a purchase cancellation be recorded? → A: Record a full
  traceable credit event and retain cancelled purchase history.
- Q: When does an installment become recognized spending? → A: It stays pending
  while its statement is Open and becomes effective on statement closing date.
- Q: How do billing-day updates affect existing statements? → A: Existing
  allocated statements stay unchanged; later purchases use updated billing days.
- Q: May an effective statement payment overdraw its paying account? → A: Yes;
  it follows ordinary expense-like account movement and may make balance negative.
- Q: May card archive retain remaining card credit? → A: No; archive is blocked
  until both outstanding obligation and card credit are zero.
- Q: How does a pending Open-statement installment appear in projections? → A:
  It counts as Expected in its current/future statement month, then Realized on
  closing without duplication.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Register and Understand a Credit Card (Priority: P1)

As an authenticated user, I want to register, review, update, and archive my
credit cards, so that I can understand each card's limit, available credit, and
past activity without treating it as a cash account.

**Why this priority**: A card's identity and limit are foundation for all
purchases, statements, and obligations.

**Independent Test**: A signed-in user creates a card with valid billing dates
and limit, reviews used and available credit, updates its descriptive details,
then archives it while retaining access to historical information.

**Acceptance Scenarios**:

1. **Given** an authenticated user has no card named `Nubank`, **When** they
   create one with a positive R$ 5.000,00 limit, closing day 10, due day 17,
   and valid optional appearance details, **Then** it belongs only to them,
   appears active, and initially has R$ 0,00 used plus R$ 5.000,00 available.
2. **Given** a card has recorded activity, **When** its owner views card
   details, **Then** they can identify its institution, optional last-four
   identifier, limit, used credit, available credit, status, and history.
3. **Given** a user enters a zero, negative, malformed, over-precision, or
   out-of-range limit; a day outside 1 through 31; or an unsupported visual
   value, **When** they save, **Then** no card is created or changed and clear
   field-level feedback explains correction needed.
4. **Given** a card has no outstanding obligation and R$ 0,00 card credit,
   **When** its owner archives it, **Then** it no longer accepts new purchases
   but historical purchases, statements, and payments remain readable.
5. **Given** a card has outstanding obligations, **When** its owner attempts
   to archive it, **Then** archiving is rejected with the outstanding amount
   and instruction to settle it first.
6. **Given** an active card has no purchases, payments, or credit events in its
   current cycle, **When** its owner views statements, **Then** current Open
   statement shows R$ 0,00 plus its billing period, closing date, and due date.
7. **Given** owner updates card closing or due day after a purchase is
   allocated, **When** update succeeds, **Then** existing statements and their
   installment assignments retain original dates while later purchases follow
   updated billing schedule.
8. **Given** a card has R$ 200,00 card credit and no outstanding obligation,
   **When** its owner attempts to archive it, **Then** archive is rejected with
   credit amount and instruction to resolve credit first.

---

### User Story 2 - Record Card Purchases and Installments (Priority: P1)

As an authenticated user, I want to record single-payment and installment card
purchases with expense categories, so that I can see what I owe and when each
part will be billed without reducing a financial-account balance at purchase
time.

**Why this priority**: Purchases create the obligation users need to manage;
misstating their financial effect would make every later total untrustworthy.

**Independent Test**: A user records one R$ 200,00 single-payment purchase and
one R$ 1.200,00 purchase in six installments, then verifies statements,
installments, credit use, category attribution, and unchanged cash-account
balance.

**Acceptance Scenarios**:

1. **Given** a user owns an active card and has an eligible expense category,
   **When** they record a R$ 200,00 purchase, **Then** purchase keeps its
   description, date, category, optional note, associated statement, and due
   date, while no financial-account balance changes.
2. **Given** a user records R$ 1.200,00 in six interest-free installments,
   **When** it is accepted, **Then** six sequential R$ 200,00 installments are
   created, each is assigned to its billing statement, and their sum is exactly
   R$ 1.200,00.
3. **Given** a R$ 100,00 purchase is divided into three installments, **When**
   installments are created, **Then** all are expressed to centavo precision,
   first installments receive any unavoidable remainder of one centavo, and
   their total is exactly R$ 100,00.
4. **Given** user selects another user's card or custom category, an archived
   card/category, a non-expense category, invalid amount/date, or installment
   count outside 1 through 360, **When** purchase is submitted, **Then** no
   purchase or obligation is created and clear feedback is shown.
5. **Given** a purchase would exceed available credit, **When** user submits
   it, **Then** system accepts it only after a clear over-limit warning and
   confirmation, labels card as over limit, and preserves exact negative
   available-credit amount.
6. **Given** an installment is allocated to an Open statement, **When** user
   views financial projections before its closing date, **Then** it is pending
   rather than realized; when statement closes, it becomes effective and is
   recognized in that statement-closing calendar month.

---

### User Story 3 - Review Statements and Pay Them (Priority: P1)

As an authenticated user, I want to review current and historical statements
and make full or partial payments from my financial accounts, so that I can
settle card obligations without recording same spending twice.

**Why this priority**: Statement payment is cash settlement, and must be
correctly linked to both card obligation and paying account.

**Independent Test**: A user with a closed R$ 600,00 statement makes an
effective R$ 200,00 payment from an owned financial account, reviews remaining
R$ 400,00 and payment history, then pays balance and confirms statement is
Paid without duplicate expense in financial totals or budgets.

**Acceptance Scenarios**:

1. **Given** a card has billed installments, **When** owner views statement,
   **Then** it shows card, inclusive billing period, closing date, due date,
   line items, original total, paid amount, outstanding amount, and status.
2. **Given** a closed R$ 600,00 statement and an owned active financial account
   with sufficient balance, **When** user pays R$ 200,00, **Then** account
   balance decreases by R$ 200,00 under financial-account rules, statement
   shows R$ 200,00 paid and R$ 400,00 outstanding, and status is Partially paid.
3. **Given** a statement's outstanding amount is R$ 400,00, **When** user pays
   R$ 400,00, **Then** outstanding becomes R$ 0,00, status becomes Paid, payment
   appears once in history, and available credit updates consistently.
4. **Given** a user enters no account, another user's or archived account, an
   invalid date/amount, a non-positive amount, or amount above outstanding,
   **When** payment is submitted, **Then** neither account balance nor statement
   changes and user receives correction feedback.
5. **Given** a payment is pending under established transaction rules, **When**
   it is viewed, **Then** it is clearly marked pending and does not reduce the
   account balance, statement outstanding amount, or used credit until it
   becomes financially effective.
6. **Given** a financially effective payment is edited, removed, restored, or
   changed between financial accounts, **When** change succeeds, **Then** prior
   and new account balances plus statement paid/outstanding values are restated
   exactly once and payment history remains traceable.
7. **Given** card closes on day 25 and due day is 5, **When** its March cycle
   closes, **Then** due date is April 5; if due day is 27, it is March 27.
8. **Given** an owned active financial account has R$ 50,00 and a closed
   statement has R$ 100,00 outstanding, **When** owner makes effective R$ 100,00
   payment, **Then** payment succeeds, account balance becomes R$ -50,00, and
   statement outstanding becomes R$ 0,00.

---

### User Story 4 - Monitor Obligations in Financial Views (Priority: P2)

As an authenticated user, I want card spending, obligations, and upcoming due
dates reflected consistently in my budget and dashboard views, so that I can
make decisions without double counting a purchase and its payment.

**Why this priority**: Cards have value only if their spending and liability
remain understandable beside existing finances.

**Independent Test**: A user with card purchases, installment obligations, a
budget, financial accounts, and a statement payment compares financial views
and confirms recognized spending appears once, cash balances exclude card debt,
and outstanding debt remains separately labelled.

**Acceptance Scenarios**:

1. **Given** a user has credit-card obligations, **When** they view dashboard
   information, **Then** current financial-account balances and outstanding card
   obligations are distinct, upcoming due statements are identifiable, and no
   combined figure implies debt is cash on hand.
2. **Given** a card purchase or installment has already been recognized as
   spending, **When** its statement payment becomes effective, **Then** payment
   reduces only paying-account balance and card obligation; it does not add a
   second expense to dashboard, category totals, budgets, financial history, or
   future reports.
3. **Given** a recurring expense would be charged to a credit card, **When**
   user manages recurring transactions in this release, **Then** they are told
   recurring card charges are not yet supported and can use an ordinary manual
   card purchase instead.
4. **Given** an installment belongs to an Open statement closing this or a
   future month, **When** user views that month's budget or dashboard projection,
   **Then** it appears as Expected; after closing, same net installment appears
   once as Realized and no longer as Expected.

---

### User Story 5 - Correct Recent Purchases (Priority: P3)

As an authenticated user, I want to correct an unbilled card purchase, so that
an input mistake does not propagate into future statements and budgets.

**Why this priority**: Safe correction protects trust while closed-statement
history stays stable.

**Independent Test**: A user edits a purchase before its statement closes and
verifies the revised amount, card/category/date, installment allocation, credit
use, and affected financial views; a user cannot silently alter a closed item.

**Acceptance Scenarios**:

1. **Given** every installment of a purchase remains in open statements,
   **When** owner corrects its description, note, date, amount, category, card,
   or installment count with valid values, **Then** affected open statements,
   installment allocation, credit use, and recognized spending restate without
   duplicate or lost money.
2. **Given** any installment belongs to a closed statement, **When** owner
   corrects or refunds original purchase, **Then** original purchase remains
   intact and a traceable credit event records full or partial adjustment,
   restates affected obligation and recognized spending, and never duplicates or
   silently deletes history.
3. **Given** a user has already fully paid a statement containing an installment
   later refunded by its issuer, **When** they record the refund, **Then** it
   creates a card credit rather than a cash-account movement, lowers used credit,
   preserves paid history, and makes resulting credit visible on card and
   affected statement.
4. **Given** a R$ 5.000,00-limit card has no unpaid obligations and R$ 200,00
   card credit, **When** owner views it, **Then** limit remains R$ 5.000,00,
   card credit remains separately labelled R$ 200,00, and available credit is
   R$ 5.200,00.
5. **Given** card has R$ 200,00 credit and closed unpaid statements of R$ 150,00
   then R$ 100,00, **When** credit is available, **Then** oldest statement is
   automatically Paid, next statement has R$ 50,00 outstanding, and no cash
   account is changed.
6. **Given** a purchase and all its installments are still in Open statements,
   **When** owner cancels it, **Then** full traceable cancellation credit event
   reverses its obligation and recognized spending, retains original purchase
   history, and creates no financial-account cash movement.

### Edge Cases

- Closing or due day exceeds days in selected month; it uses that month's final
  calendar day, including February and leap years.
- Due day is earlier than, equal to, or later than closing day; it is always the
  first configured due-day occurrence after that cycle's closing date.
- Purchase occurs on closing date, across year end, or in a month shorter than
  card's configured day.
- Closing or due day changes after purchases exist: allocated statements and
  installment assignments retain original schedule; only later purchases use it.
- First installment is allocated by purchase cycle and later installments move
  to each subsequent monthly cycle, including February transitions.
- An installment remains pending while its allocated statement is Open, then
  becomes effective on that statement's closing date and is recognized in that
  closing calendar month.
- Pending installment in its current/future statement-closing month is Expected;
  closing moves same net amount to Realized exactly once.
- A statement reaches closing, is unpaid on following due date, is partially
  paid, fully paid, or receives a pending/removed/restored payment.
- Effective payment may reduce an owned paying-account balance below zero; it
  follows ordinary expense-like account movement, not Transfer source rules.
- An active card's current cycle has no activity: it remains visible as Open
  R$ 0,00; archived card receives no new current-cycle statement.
- Card has zero outstanding obligation but positive card credit: archive stays
  blocked until card credit is zero.
- Purchase uses more than card's available credit, has a zero-centavo rounding
  remainder, or installment count is one.
- Card, linked account, or category is archived after historical association.
- A category is later archived; historical purchases and recognized spending
  retain readable category identity.
- Purchase is corrected while open; correction moves between statements/cards,
  changes installment count, or conflicts with card availability.
- A purchase is cancelled before or after statement closing: cancellation is a
  full traceable credit event, never a permanent deletion.
- User attempts access to another user's cards, purchases, installments,
  statements, payments, accounts, categories, or aggregated information.
- User uses keyboard only, assistive technology, high zoom, narrow layout, or
  Light, Dark, or System theme; all states and money meaning remain clear
  without color alone.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST let authenticated users create, list, view, update,
  and archive only credit cards they own. Every card belongs to exactly one user
  and is distinct from a financial account.
- **FR-002**: A card MUST record name, financial institution, positive credit
  limit, closing day, due day, and active/archived status; it MAY record a
  last-four identifier plus color or icon. It MUST NOT request, retain, or show
  full card number, CVV, PIN, or other payment credentials.
- **FR-003**: Credit limit MUST be a positive BRL amount from R$ 0,01 through
  R$ 999.999.999,99 with at most two decimals. Closing and due days MUST each
  be whole values from 1 through 31 and MAY be equal.
- **FR-004**: Used credit MUST equal card's net unpaid purchase principal,
  including every unpaid installment and unbilled purchase amount after
  financially effective statement payments and applied credit-event adjustments;
  it MUST never be below zero. Card credit is remaining effective credit-event
  value after it has reduced affected unpaid obligation. Available credit MUST
  equal stated limit minus used credit plus card credit; it MAY exceed stated
  limit, while card credit and stated limit remain separately labelled. Full
  installment purchase amount, rather than only current installment, consumes
  limit when recorded.
- **FR-005**: A purchase exceeding available credit MUST first return a typed
  confirmation-required outcome with the exact resulting negative available
  credit. The user may resubmit the unchanged purchase with explicit
  confirmation; it then remains permitted and identifies card as over limit.
  Confirmation does not change limit or create a cash-account movement.
- **FR-006**: Only active owned cards may accept new purchases. Archived cards
  remain readable with their historical purchases, installments, statements,
  payments, and credit figures, but cannot accept new purchases or be archived
  while any statement has outstanding amount or card credit remains positive.
- **FR-007**: A credit-card purchase MUST record an owned card, non-empty
  description, purchase date from 1900-01-01 through 2100-12-31, positive BRL
  total, valid owned expense category, optional note, and installment count.
  Initial version supports interest-free installments only.
- **FR-008**: A purchase MUST NOT directly alter a checking, savings, cash, or
  other ordinary financial-account balance. It MUST create card obligation and
  statement installment entries only.
- **FR-009**: Single-payment purchases produce one installment equal to purchase
  total. Installment purchases produce consecutive sequence numbers and positive
  centavo-precise amounts whose exact sum equals purchase total; any indivisible
  centavos are distributed one at a time to earliest installments.
- **FR-010**: System MUST determine each installment's statement from card's
  billing cycle and preserve an explanation of purchase date, cycle period,
  closing date, and due date for user review. A purchase made on closing date
  belongs to statement closing that day. A statement remains Open through its
  inclusive closing calendar day and finalizes at the start of the next
  America/Sao_Paulo calendar date.
- **FR-011**: For each cycle, closing and due day MUST be calendar-aware: if a
  configured day does not exist in a month, use that month's last calendar day.
  Due date MUST be first configured due-day occurrence after closing date: it is
  in closing month when configured due day is later than actual closing day and
  next month otherwise. Cycle calculation MUST remain predictable across
  month/year boundaries, February, and leap years.
- **FR-011A**: Updating card closing or due day MUST NOT change an existing
  allocated statement's period, closing date, due date, or installments. Only
  purchases recorded after update use new billing-day configuration.
- **FR-012**: A statement MUST identify card, inclusive billing period, closing
  date, due date, line items, original amount, total paid, outstanding amount,
  credit-event adjustments, resulting net amount, any resulting card credit,
  and status. Every active card MUST show its current Open statement, including
  R$ 0,00 statement with no activity. Statements remain readable after
  cards/categories/accounts are archived; archived cards receive no new current
  statement.
- **FR-013**: A statement is Open through its inclusive closing date; Closed
  from the first America/Sao_Paulo calendar date after closing with no effective
  payment or applied card credit; Partially paid after total
  effective payment plus applied card credit is greater than zero while
  outstanding remains positive; Paid when outstanding is zero; and Overdue when
  closed, outstanding is positive, and business date is after due date. A paid
  statement stays Paid even after due date. Automatic card-credit application
  uses these same outstanding and status rules.
- **FR-014**: Only Closed, Partially paid, or Overdue statements may receive
  payment. One payment MUST be tied to exactly one statement and one owned
  financial account; full and partial payments are supported. Payment amount
  MUST be positive and no greater than current outstanding amount.
- **FR-015**: A financially effective statement payment MUST reduce paying
  financial-account balance under established account/transaction rules and
  reduce statement outstanding amount by same amount. Pending payment MUST do
  neither until financially effective. Duplicate submission or repeated effect
  of one payment MUST be prevented. Effective payment MAY reduce paying-account
  balance below zero; insufficient-balance rejection does not apply because this
  is expense-like settlement, not an account transfer.
- **FR-016**: Editing, removing, restoring, correcting, or changing financial
  account for statement payment MUST restate card obligation and every affected
  financial-account balance exactly once, retain payment history, and respect
  established transaction lifecycle rules. Archived accounts remain visible for
  historical payment; they are unavailable for a new or changed payment.
- **FR-017**: Card purchases and installments MUST use only valid expense
  categories available to owner. Another user's category, an unavailable custom
  category, income category, or newly selected archived category MUST be
  rejected. Historical category identity remains readable after archive.
- **FR-018**: Each installment MUST consume category spending and monthly budget
  in calendar month containing its associated statement closing date. It MUST
  remain pending while that statement is Open and become effective on statement
  closing date. Pending installment MUST contribute Expected spending in its
  current/future statement-closing month; closing moves same net installment to
  Realized exactly once. Each effective recognized installment MUST contribute
  once only to dashboard, category spending, budget, financial history, and
  future reports; neither original purchase total nor statement payment may
  create a duplicate recognized expense.
- **FR-019**: A statement payment MUST be a settlement transfer of cash from a
  financial account to card obligation, not a new categorized expense. It MUST
  not independently consume a category budget or duplicate spending already
  recognized from its purchase/installments.
- **FR-020**: Dashboard integration MUST separately present current financial
  account balances, outstanding card obligations, upcoming due statements, and
  available credit where cards exist. It MUST not redesign unrelated dashboard
  sections or double count a purchase/payment pair.
- **FR-021**: Initial version MUST not add a separate recurring-card mechanism.
  Existing recurring transactions remain financial-account based; recurring card
  purchases are excluded while preserving future ability to add them.
- **FR-022**: A purchase can be corrected directly only while all its
  installments are in Open statements. A cancellation at any lifecycle stage
  MUST be a full traceable credit event, never permanent deletion. Initial
  version MUST support traceable full and partial refunds plus post-closing
  corrections as credit events; original purchase history remains intact. Each
  credit event MUST identify its source purchase, amount, date, reason, and
  affected statement(s), and MUST
  restate affected statement amount, outstanding obligation, used/available
  credit, and installment-period recognized spending exactly once. Credit-event
  amount MUST be positive, no more than uncredited source-purchase amount, and
  allocated in original installment sequence; centavo rounding follows same
  earliest-installment rule as purchase allocation. Credit first reduces unpaid
  affected statement amount; any excess after that statement is paid becomes
  visible card credit, never a negative outstanding amount. Available card
  credit MUST automatically reduce oldest unpaid statement's outstanding amount
  and continue in due-date order until exhausted; any remainder is retained for
  next statement. A credit event or automatic card-credit application MUST NOT
  create or erase a financial-account cash movement by itself.
- **FR-023**: System MUST clearly communicate validation and state conflicts for
  invalid cards, limits, billing dates, purchases, categories, installment
  counts, card ownership, payment accounts, payment amounts, duplicate payments,
  closed-purchase corrections, archive requests, and invalid state transitions.
- **FR-024**: Users may access only their own cards, purchases, installments,
  statements, payments, paying accounts, categories, and credit/budget/dashboard
  information. Unauthorized resources must not reveal existence or details.
- **FR-025**: Financial values, statement/payment statuses, due dates, unpaid
  amounts, over-limit state, installment sequences, and success/error feedback
  MUST be clear in responsive Light, Dark, and System themes at narrow widths,
  keyboard use, assistive technology, and 200% zoom without relying on color.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Every changed action MUST enforce authentication, owner-scoped
  authorization, valid lifecycle state, and clear input validation. No sensitive
  card credentials are collected, retained, exposed, or logged.
- **SQR-002**: Automated coverage MUST prove owner isolation; exact centavo and
  installment arithmetic; billing-date edge cases; statement transitions;
  payment lifecycle; financial-account restatement; no duplicate expenses;
  category/budget/dashboard recognition; and critical user journeys.
- **SQR-003**: This feature has no uploads, external issuer connection, real
  payment processing, or new secrets. Existing financial data handling and
  access protections apply to all card data.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define credit-card, purchase,
   installment, statement, and payment behavior; authorization; validation;
   financial-account, category, budget, and dashboard recognition rules; and
   automated coverage before frontend work begins.
2. **Frontend** (`../zunera-frontend`): After backend behavior is available,
   provide card management, purchase entry, statement/payment review, history,
   dashboard presentation, accessible feedback, responsive themes, and coverage
   that consume established backend behavior.

### Key Entities *(include if feature involves data)*

- **Credit Card**: User-owned card identity, issuer, optional display identity,
  stated limit, billing-day rules, status, used credit, card credit, and
  available credit.
- **Credit Card Purchase**: Categorized user-entered obligation, with purchase
  details, total, and one or more installment obligations.
- **Installment**: A numbered centavo-precise part of purchase allocated to one
  statement, retaining its recognized-spending relationship.
- **Credit Card Statement**: Card's calendar-aware billing cycle and its line
  items, original total, payments, outstanding amount, due date, and status.
- **Statement Payment**: Traceable settlement of one statement from one owned
  financial account, with amount, payment date, note, and financial state.
- **Credit Event**: Traceable cancellation, full or partial refund, or
  post-closing correction tied to original purchase and installments; reduces
  obligation or creates card credit, without directly moving money in a
  financial account.
- **Financial Recognition**: Rule that keeps an installment pending while its
  statement is Open and makes it effective in that statement's closing calendar
  month. It is Expected in current/future closing month while pending, then
  Realized once when effective; this determines its single entry into category
  spending, budgets, dashboard, history, and reports, distinct from obligation
  creation and cash settlement.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can create a valid card and record a single-payment or
  six-installment purchase in under 3 minutes, with 100% of recorded
  installment totals matching original purchase total to centavo.
- **SC-002**: In 100 representative statement-cycle cases covering closing-day,
  month-end, February, leap-year, and year-boundary dates, every installment
  has one predictable statement, closing date, and due date.
- **SC-003**: In 100 representative payment lifecycle cases, financially
  effective payments change paying account and statement outstanding by equal
  amounts exactly once, while pending/removed payments have no unintended
  current financial effect.
- **SC-004**: In 100 representative purchase-and-payment pairs, recognized
  category spending and budget consumption count each effective closing-month
  installment once and statement payment zero times as a new expense.
- **SC-005**: At least 95% of moderated users can identify current outstanding
  obligation, next due date, statement status, and available credit without
  assistance in accessible light and dark layouts.

## Assumptions

- Users manage personal BRL cards manually; no issuer integration, automatic
  import, real payment processing, shared cards, multi-currency, rewards,
  cashback, interest, revolving credit, refinancing, or debt negotiation is
  included.
- Card identifier is optional and limited to non-sensitive display information,
  such as last four digits. Full credentials are never needed.
- Financial-account and transaction lifecycle rules already define effective,
  pending, removed, restored, and archived-account behavior; card payments
  follow those rules while retaining card-specific obligation meaning.
- Effective statement payment follows ordinary expense-like account movement and
  may overdraw its selected active account; it does not use Transfer's
  insufficient-source-balance restriction.
- Used credit follows full remaining purchase principal, so full installment
  purchase consumes limit at recording and effective payment restores limit by
  its settled amount. Refund-created card credit is separately shown and adds
  usable capacity; available credit may therefore exceed stated limit.
- Partial statement payments are in scope; no interest, late fee, revolving
  balance, or financing calculation is added for unpaid amounts.
- Over-limit purchases are allowed only with explicit warning confirmation;
  issuer authorization is not implied or performed.
- Existing Categories, Monthly Budgets, Financial Accounts, Transactions, and
  Dashboard features remain source of their own established meanings; this
  feature extends them only through stated credit-card recognition and settlement
  rules.
- Cancellations, post-closing full/partial refunds, and corrections are
  traceable credit events, not silent edits or deletions. They adjust affected
  card obligations and recognized installment-period spending but do not
  independently move cash.
