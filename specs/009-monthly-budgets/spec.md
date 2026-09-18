# Feature Specification: Monthly Budgets

**Feature Branch**: `009-monthly-budgets`  
**Backend Branch**: `009-monthly-budgets` (`../zunera-backend`)  
**Frontend Branch**: `009-monthly-budgets` (`../zunera-frontend`)  
**Created**: 2026-09-18  
**Status**: Ready for implementation

**Input**: User description: "Create Budgets feature for Zunera: monthly,
expense-category planning with planned, realized, available, expected, and
projected spending."

## Clarifications

### Session 2026-09-18

- Q: When a budget exists but has no category plans, how should monthly
  utilization appear? → A: Show planned, realized, and available as R$ 0,00;
  show utilization, status, and progress as Not applicable.
- Q: If source budget contains category plan whose category is now archived,
  what happens on copy? → A: Block whole copy and explain archived category
  must be resolved first.
- Q: For a budget month already ended, how should pending expenses appear? → A:
  Do not show expected or projected values for past months.
- Q: If an active expense category already has a budget plan but no
  transactions, may its classification change to income? → A: Reject the
  classification change once category appears in any budget plan.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create and Manage Monthly Plan (Priority: P1)

As an authenticated user, I want to create a budget for a calendar month and
set planned amounts for expense categories, so that I can set a clear spending
plan without changing my finances.

**Why this priority**: A trustworthy monthly plan is foundation for every
comparison and control capability.

**Independent Test**: A signed-in user creates one monthly budget, adds several
valid expense categories and positive planned amounts, changes an amount, then
removes one category. The budget is saved correctly while financial history and
account balances stay unchanged.

**Acceptance Scenarios**:

1. **Given** an authenticated user has no budget for September 2026 and has
   active available expense categories, **When** they create September's budget
   and add Food at R$ 1.000,00, **Then** it belongs only to them, identifies
   September 2026, and shows Food as planned at R$ 1.000,00.
2. **Given** a user is editing a budget, **When** they add a category already
   budgeted for that month, **Then** no duplicate category line is created and
   clear feedback explains that one category can appear only once per budget.
3. **Given** a Food plan of R$ 800,00 and existing effective Food expenses,
   **When** the user changes its plan to R$ 1.000,00, **Then** only the plan
   changes and available amount plus utilization are recalculated from the
   unchanged expense history.
4. **Given** a budget contains Food and Food expenses exist that month, **When**
   the user removes Food from the budget, **Then** the category, transactions,
   and account balances remain unchanged and those expenses are shown as
   unbudgeted for that month.
5. **Given** a user enters zero, a negative, malformed, over-precision, or
   out-of-range monetary value, **When** they save a planned amount, **Then**
   no budget change occurs and field-relevant correction feedback is shown.
6. **Given** an expense category is included in any budget plan, **When** its
   owner attempts to change its classification to income, **Then** the change is
   rejected with clear state-based feedback and every budget plan remains intact.

---

### User Story 2 - Monitor Monthly Spending (Priority: P1)

As an authenticated user, I want to see planned, realized, and available
spending for each budgeted category and for the whole month, so that I know
whether I can stay within my plan.

**Why this priority**: The feature is useful only when planned spending can be
reliably compared with financial reality.

**Independent Test**: A signed-in user with effective expense, income,
transfer, and pending transactions in one month views their budget and confirms
that only matching effective expenses consume category and monthly budget
amounts.

**Acceptance Scenarios**:

1. **Given** Food is planned at R$ 1.000,00 and effective Food expenses total
   R$ 720,00 in selected month, **When** the user views the budget, **Then**
   Food shows R$ 720,00 realized, R$ 280,00 available, and 72% used.
2. **Given** Food is planned at R$ 1.000,00 and effective Food expenses total
   R$ 1.070,00, **When** the user views the budget, **Then** it communicates
   that Food is exceeded, including R$ 70,00 over plan, rather than relying on
   an unexplained negative value or color alone.
3. **Given** a selected month contains income, own-account transfers, pending
   expenses, removed expenses, and effective expense transactions, **When** the
   budget is viewed, **Then** only non-removed effective expenses with the
   matching category and transaction date contribute to realized spending.
4. **Given** a user changes, removes, restores, reclassifies, re-categorizes,
   re-dates, or changes the financial state of a relevant transaction, **When**
   the budget is next shown, **Then** each affected monthly and category total
   reflects the authoritative transaction history without duplicating or
   preserving a stale realized amount.
5. **Given** selected month has expenses in categories not budgeted that month,
   **When** the user views its summary, **Then** budgeted expenses,
   unbudgeted expenses, and total expenses are separately labelled and shown.

---

### User Story 3 - Review Months and Copy Plan (Priority: P2)

As an authenticated user, I want to navigate months, review prior budgets, and
copy a prior plan into another month, so that I can reuse useful plans without
changing past financial records.

**Why this priority**: Reusing a month lowers planning effort while preserving
the meaning of historical plans.

**Independent Test**: A user creates September budget, navigates to it after
September ends, copies its category amounts to October, then changes October
without changing September or any transactions.

**Acceptance Scenarios**:

1. **Given** a user views one month, **When** they navigate to previous or next
   month, **Then** selected month and year are explicit and that month's budget
   is shown independently.
2. **Given** no budget exists for selected month, **When** user opens it,
   **Then** a useful empty state confirms no plan exists and offers Create
   Budget plus Copy Previous Budget when an eligible owned source budget exists.
3. **Given** September contains category plans and realized expenses, **When**
   user copies it to empty October, **Then** October receives independent
   category planned amounts only; it receives no realized amounts, expected
   amounts, transactions, financial history, or account-balance changes.
4. **Given** source September later changes after successful copy to October,
   **When** user reviews October, **Then** October keeps its independently
   copied planned amounts unless user changes October directly.
5. **Given** a user views a past budget containing an archived category,
   **When** the historical budget is opened, **Then** its planned amount,
   realized spending, available amount, and status remain understandable.
6. **Given** destination month already has a budget, **When** user attempts to
   copy another month into it, **Then** copy is rejected with clear feedback and
   destination budget remains unchanged.
7. **Given** a budget exists with no category plans, **When** user views its
   monthly summary, **Then** planned, realized, and available show R$ 0,00 and
   utilization, status, and progress show Not applicable.
8. **Given** source budget contains a plan for archived category, **When** user
   attempts to copy source budget, **Then** copy is rejected, destination is
   unchanged, and clear feedback identifies archived category as reason.

---

### User Story 4 - Anticipate Future Expenses (Priority: P3)

As an authenticated user, I want expected spending distinguished from realized
spending, so that I can recognize a likely budget issue before it becomes an
effective financial expense.

**Why this priority**: Projection adds foresight but must never weaken the
trustworthiness of actual spending.

**Independent Test**: A user with a planned Food amount, effective Food
expenses, and eligible future expense information can distinguish realized from
expected values and verify projected values never change balances or actual
available amount.

**Acceptance Scenarios**:

1. **Given** Food is planned at R$ 1.000,00, realized Food expenses are
   R$ 550,00, and expected Food expenses are R$ 200,00, **When** the budget is
   viewed, **Then** it separately shows R$ 550,00 realized, R$ 200,00
   expected, R$ 750,00 projected spending, and R$ 250,00 projected available.
2. **Given** projected spending exceeds a category plan while realized spending
   does not, **When** user views category, **Then** it is described as projected
   to exceed plan, not as already exceeded.
3. **Given** no eligible expected expense information exists, **When** budget
   is viewed, **Then** realized values remain complete and an empty or omitted
   projection area does not imply no future expenses will occur.
4. **Given** selected budget month has ended and contains pending expenses,
   **When** user views its budget, **Then** expected and projected spending are
   not shown and realized spending remains limited to effective expenses.

### Edge Cases

- A user has no budget for selected month, or a budget exists with no category
  plans, no expenses, or no expected spending.
- A budget with no category plans has total planned, realized, and available of
  R$ 0,00; its overall utilization, status, and progress are Not applicable.
- Selected month is in past, current, or future; month/year remain unambiguous
  at year boundaries.
- A past budget month contains pending expense transactions. They remain outside
  realized totals and expected/projected values are not shown for that month.
- A category has no realized spending, exactly reaches its plan, reaches 80% of
  plan, exceeds its plan, or has a plan so small that rounded percentage display
  requires a precise underlying calculation.
- A single effective expense changes amount, category, date, state, or removal
  status; all old and new affected month/category figures must restate.
- Income, transfer, removed, pending, or a different user's transaction shares
  date/category-like data with a budgeted expense.
- A source budget has zero categories; source/destination month is same; source
  is future or historical; or destination contains a budget.
- A source budget has one or more plans for archived categories; copy is blocked
  and destination remains unchanged, including when it otherwise has no budget.
- Category is archived after a plan exists, restored later, or has expenses in
  both budgeted and unbudgeted months.
- An expense category is budgeted before it has any transaction, then its owner
  attempts to reclassify it as income; the classification change is rejected.
- User operates with keyboard only, assistive technology, high zoom, narrow
  screen, Light, Dark, or System theme.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST let authenticated users create, view, update, and
  copy only monthly budgets they own, and MUST prevent all access to another
  user's budgets, budget items, categories, and financial calculations.
- **FR-002**: A budget MUST belong to exactly one user and exactly one calendar
  month/year. A user MUST have at most one budget for a given month/year.
- **FR-003**: A monthly budget MUST contain zero or more category plans; each
  category plan belongs to one budget and may occur only once in it.
- **FR-004**: Creating, copying, changing, or removing a budget or category plan
  MUST never create, edit, remove, or change state of financial transactions,
  change account balances, or otherwise move money.
- **FR-005**: A new category plan MUST use an active expense category available
  to user. Income categories, unavailable personal categories, another user's
  personal category, and archived categories MUST be rejected as new plans.
- **FR-005A**: Once a category appears in any budget plan, system MUST reject a
  classification change that would make it non-expense, without changing the
  category or any budget plan. This category-plan association locks
  classification even when the category has no financial transaction.
- **FR-006**: A planned amount MUST be a positive Brazilian-real value from
  R$ 0,01 through R$ 999.999.999,99 with at most two decimals. Zero, negative,
  malformed, over-precision, and out-of-range values MUST be rejected without
  financial-precision loss.
- **FR-007**: Budget period selection and navigation MUST use a named calendar
  month/year. When no budget exists, system MUST show no fabricated plan or
  zero-spending claim and offer appropriate creation/copy next actions.
- **FR-008**: For each category plan, realized spending MUST equal sum of its
  owner's non-removed financially effective expense transactions whose
  category and transaction date fall within budget month. Income, transfers,
  pending/non-effective, and removed transactions MUST NOT consume budget.
- **FR-009**: Realized spending MUST be derived at viewing time from transaction
  history, not independently maintained, and MUST restate after every relevant
  transaction lifecycle or financial-data change.
- **FR-010**: Category available amount MUST equal planned amount minus realized
  spending. It MAY be below zero; interface MUST identify excess amount and
  exceeded state in words and values, not by color alone.
- **FR-011**: Category utilization MUST equal realized spending divided by
  planned amount times 100. It MAY exceed 100%; display MUST communicate this
  accurately without misleading rounding.
- **FR-012**: Monthly summary MUST show total planned, realized spending for
  budgeted categories, actual available amount, and overall utilization using
  those same category-plan values. It MUST separately show unbudgeted effective
  expense spending and total effective expense spending for selected month. If
  budget has no category plans, total planned, realized, and available MUST be
  R$ 0,00, while overall utilization, status, and progress MUST be shown as Not
  applicable rather than zero percent or Within budget.
- **FR-013**: System MUST classify a category as Within budget below 80% used,
  Approaching planned amount from 80% through less than 100%, Budget reached at
  exactly 100%, or Budget exceeded above 100%. Text, numeric values, and a
  non-color cue MUST communicate status.
- **FR-014**: Existing plan associated with later archived category MUST remain
  visible and understandable in current and historical budgets, using its
  preserved category identity and an archived-state label. It MUST be read-only
  and MUST NOT become a normal new-plan option, be edited, or be removed unless
  category is first restored under Category Management rules.
- **FR-015**: Removing a category plan MUST remove only its plan from that one
  monthly budget. It MUST leave category and transactions intact and reclassify
  that month's qualifying expenses as unbudgeted in budget summary.
- **FR-016**: User MUST be able to view prior and future months independently.
  Editing a budget MUST change only its own planned definition; it MUST not
  rewrite financial history or definitions in any other month.
- **FR-017**: Copy source budget MUST copy only source category planned amounts
  into independent destination budget. It MUST not copy realized/expected or
  projected figures, transactions, category lifecycle changes, or balances.
- **FR-018**: Copy source and destination MUST belong to same user and be
  different calendar months. If destination already has a budget, system MUST
  reject copy with clear feedback and leave every destination plan unchanged.
  If source contains a plan whose category is archived, system MUST reject whole
  copy with clear feedback and leave destination unchanged; it MUST NOT omit
  archived plans or create a new read-only archived plan.
- **FR-019**: Historical budget views MUST continue to show what was planned,
  actual realized spending recalculated from retained transaction history,
  available amount, utilization, category status, and archived-category
  presentation according to FR-014. Later category renames or visual-identity
  changes MUST NOT replace historical budget's preserved category identity.
- **FR-020**: Expected spending, when present, MUST be separate from realized
spending and MUST NOT alter account balances, realized utilization, or actual
available amount. It MUST equal sum of owner's non-removed pending expense
transactions with matching category and transaction date in selected current
or future budget month. Expected and projected values MUST NOT be shown for a
budget month that has ended. A recurrence definition without a pending
transaction MUST NOT count as expected spending; a generated pending
recurrence transaction qualifies as a pending expense transaction. Monthly
expected spending MUST sum expected values of budgeted category plans only;
pending expenses in unbudgeted categories remain outside this budget projection.
- **FR-021**: Projected spending MUST equal realized plus expected spending, and
projected available MUST equal planned minus projected spending. Projected
fields and projected excess state MUST be visibly labelled as projections. For
the monthly summary, projected spending, available, and status MUST use the
same budgeted-category scope and thresholds as its actual values. Current or
future summaries with category plans return zero expected spending when no
qualifying pending expense exists; ended months and zero-plan budgets omit
expected/projected summary values.
- **FR-022**: Budget information MUST remain clear in empty, error, loading,
  Light, Dark, System-theme, responsive, keyboard-only, high-zoom, and
  assistive-technology use. Visual progress must have equivalent textual values
  and status.
- **FR-023**: Dashboard summary may present budget values and attention states
  derived by Budget feature rules, but MUST not create competing calculations or
  modify budgets.
- **FR-024**: Weekly/annual/custom periods, rollover, envelope/shared/account
  budgets, income/savings allocation, goals, recommendations, automatic
  creation, synchronization, notifications, and advanced forecasting are out of
  scope.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Budget actions and calculations MUST require authentication,
  enforce user ownership, validate every input, and deny unavailable category,
  transaction, or budget data without exposing another user's information.
- **SQR-002**: Automated coverage MUST verify ownership isolation; monthly
  uniqueness; positive-money validation; category eligibility; copy behavior;
  archived-category/classification-lock behavior; state-aware realized values; projected values;
  unbudgeted totals; status thresholds; transaction-driven restatement; and
  critical accessible user journeys.
- **SQR-003**: No secret, upload, or external financial-data handling is
  introduced. Budget data and derived financial information remain visible only
  in authenticated financial-management experiences.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define protected budget behavior,
   ownership, validation, monthly and category-plan lifecycle, authoritative
   transaction-derived calculations, and automated verification before frontend
   delivery begins.
2. **Frontend** (`../zunera-frontend`): Deliver month navigation, planning,
   copy, empty states, accessible responsive progress/status presentation, and
   automated verification consuming confirmed budget behavior.

### Key Entities *(include if feature involves data)*

- **Monthly Budget**: One user's planning record for one calendar month/year;
  independent from transactions and account balances.
- **Category Plan**: One positive planned expense amount for one expense
  category within one monthly budget.
- **Realized Spending**: Derived sum of qualifying effective expense
  transactions for a category/month; never an independent financial truth.
- **Expected Spending**: Separately identified, non-effective future expense
  information eligible under clarified source rules.
- **Projected Spending**: Realized plus expected spending, presented only as a
  projection.
- **Budget Status**: Textual category state determined from utilization:
  Within, Approaching, Reached, Exceeded.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% acceptance-test users can create a month with three
  category plans in under 2 minutes without assistance and without creating a
  financial transaction or changing an account balance.
- **SC-002**: In data-integrity validation, 100% tested qualifying transaction
  changes update all affected budget totals on next view, while income,
  transfers, pending, removed, and other-user data contribute 0 to realized
  spending.
- **SC-003**: At least 90% acceptance-test users can identify planned,
  realized, available, utilization, and status for a category in under 30
  seconds without relying on color.
- **SC-004**: At least 90% acceptance-test users can identify budgeted,
  unbudgeted, and total expense spending for a month in under 30 seconds.
- **SC-005**: In copy validation, 100% copied plans retain only category planned
  amounts and leave source plan, financial history, transactions, and account
  balances unchanged.

## Assumptions

- Existing authenticated-user, category, transaction, transfer, and recurring
  transaction rules are authoritative dependencies.
- Transaction specification defines effective as settled financial state and
  pending as planned/non-effective; removed transactions do not count.
- Calendar months and financial dates follow established Zunera business-date
  conventions; money uses existing Brazilian-real precision and formatting.
- No budget in a selected month is a normal empty state, not an error or an
  implicit zero plan.
- Unbudgeted expenses are prominent in every monthly summary as separately
  labelled values, while category-level detail remains focused on planned lines.
- Initial Approaching status uses fixed 80% utilization threshold; it is not
  user-configurable in this feature.
- Archived plans are read-only and show preserved identity until category is
  restored; copy rejects an occupied destination; expected spending uses only
  pending expense transactions in selected month; budget association locks
  category classification.
