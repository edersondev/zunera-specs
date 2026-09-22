# Feature Specification: Transactions Month Navigator

**Feature Branch**: `011-transactions-month-navigator`
**Backend Branch**: `011-transactions-month-navigator` (`../zunera-backend`)
**Frontend Branch**: `011-transactions-month-navigator` (`../zunera-frontend`)
**Created**: 2026-09-22
**Status**: Ready for implementation

**Input**: User description: "In the budgets page has a function budget-month-navigator.
I want the same function in the transaction page. The line where has the search input
and the button Filter, split it into two columns, in the left will be the search input
and filter buttons and the right the month navigator."

## Clarifications

### Session 2026-09-22

- Q: What should the month navigator do on the transactions page? → A: Filter the
  list by the whole selected month (first to last day); the page opens on the current
  business month.
- Q: The income/expense/result cards read totals that ignore date filters — keep them
  all-time or scope them to the month? → A: Scope totals to the requested period so
  the cards match the month on screen.
- Q: How should the navigator code be shared with budgets? → A: Promote it to one
  shared month navigator component used by both pages.
- Q: When a custom date range from the Filters dialog is active, what should the
  arrows do? → A: The custom range stays until the user clicks an arrow; clicking one
  replaces the custom range with the whole chosen month.
- Q: What should Clear filters do with the month scope? → A: Keep the month shown in
  the navigator and clear only search, type, status, account, category, and custom
  period criteria.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Browse Transactions by Month (Priority: P1)

As an authenticated user, I want a month navigator in the transactions page header,
between the page title and the page actions, so that I can step through one calendar
month at a time the same way I do on the budgets page.

**Why this priority**: Month-by-month browsing is the requested capability; everything
else only improves its accuracy and presentation.

**Independent Test**: A signed-in user opens the transactions page, sees the current
month in the navigator next to the search and filter controls, and uses the previous and
next arrows to display another month's movements.

**Acceptance Scenarios**:

1. **Given** a signed-in user opens the transactions page without query parameters,
   **When** the page finishes loading, **Then** the navigator shows the current business
   month and the list shows only movements dated inside that month.
2. **Given** the navigator shows a month, **When** the user selects the next or previous
   arrow, **Then** the label advances by exactly one calendar month, including across a
   year boundary, and the list reloads for the new month.
3. **Given** the user has search text, type, status, account, or category criteria
   active, **When** they change the month, **Then** those criteria stay applied and only
   the period changes.
4. **Given** the navigator shows a month, **When** the user reloads the page or follows
   the address, **Then** the same month is restored from the address.
5. **Given** the user opens the page on a narrow screen or with keyboard only, **When**
   they use the navigator, **Then** the control remains reachable, announces the month,
   and the header stacks the navigator with the heading and page actions without
   overlap.

---

### User Story 2 - Month Totals Match the Visible Movements (Priority: P2)

As an authenticated user, I want the realized income, expense, and result cards to
describe the month I am viewing, so that the summary and the list tell the same story.

**Why this priority**: Month navigation that leaves all-time totals on screen would
report figures that contradict the movements below them.

**Independent Test**: A user with movements in two different months switches between
them and sees the cards change to the totals of the visible month, including at both
range boundaries.

**Acceptance Scenarios**:

1. **Given** a user has effective income and expense transactions inside and outside the
   selected month, **When** the month is displayed, **Then** the cards count only the
   movements dated inside that month.
2. **Given** a movement is dated exactly on the first or last day of the month, **When**
   totals are computed, **Then** that movement is included.
3. **Given** a month contains no qualifying movement, **When** the month is displayed,
   **Then** all three cards report zero instead of another month's figures.
4. **Given** a pending or removed movement inside the month, **When** totals are
   computed, **Then** it is excluded, and transfers remain excluded from the cards.
5. **Given** a request for history without a period, **When** totals are computed,
   **Then** the previous all-time behavior is preserved.

---

### User Story 3 - One Month Control Across Screens (Priority: P3)

As a user, I want the budgets page to keep behaving exactly as before while both pages
share a single month control, so that month navigation feels consistent and does not
regress.

**Why this priority**: Sharing the control prevents two drifting implementations, but
budgets behavior must not change.

**Independent Test**: The budgets page month control still shows the selected month,
still moves across year boundaries, still disables while loading, and still passes its
existing keyboard and screen-reader checks.

**Acceptance Scenarios**:

1. **Given** the budgets page, **When** the user navigates months, **Then** the behavior
   and wording are unchanged from before this feature.

---

### Edge Cases

- Navigating from December to January and back crosses the year boundary correctly.
- February, including a leap year, resolves the last day correctly.
- A custom date range applied in the Filters dialog stays visible until the user clicks
  an arrow, which then replaces it with the whole chosen month.
- A custom range that is not a whole month keeps the navigator label on the month of its
  first day.
- Clearing filters keeps the visible month and only drops the other criteria.
- The empty state for a month with no movements distinguishes "nothing recorded yet"
  from "nothing matches the current criteria".
- The removed-transactions screen keeps its current behavior and does not gain a month
  navigator.
- A history request without a period keeps the previous all-time totals behavior.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The transactions page MUST present a month navigator in the page header,
  positioned between the page title block and the page actions, and the search and filter
  controls MUST stay together in the filter row below it.
- **FR-002**: The navigator MUST show the selected month in the user's locale and MUST
  provide previous and next actions that move exactly one calendar month, including
  across year boundaries.
- **FR-003**: Opening the transactions page without a period MUST default to the current
  business month and MUST show only movements dated inside it.
- **FR-004**: The selected month MUST be expressed as an inclusive period from the first
  to the last day of that month when the movement list is requested.
- **FR-005**: Changing the month MUST preserve the user's search, type, status, account,
  and category criteria, MUST restart pagination, and MUST be reflected in the address so
  reload, back, and shared links restore the same month.
- **FR-006**: A custom date range chosen in the filters dialog MUST take precedence over
  the navigator label until the user changes the month, at which point the whole month
  replaces the custom range.
- **FR-007**: The period criterion MUST be presented as an active criterion only while it
  differs from the navigator's month, and removing it MUST return to the navigator's
  month.
- **FR-008**: Clear filters MUST return the period to the navigator's month and MUST
  clear the remaining criteria.
- **FR-009**: The transactions page MUST treat the navigator month as the baseline scope,
  so an account without movements keeps showing the "nothing recorded yet" empty state
  instead of the filtered-empty message.
- **FR-010**: History responses MUST report income, expense, and net result totals for the
  requested period only, counting effective, non-removed income and expense movements
  whose date falls inside the inclusive range.
- **FR-011**: When a history request omits the period, totals MUST keep their previous
  all-time behavior, and transfers MUST stay excluded from totals.
- **FR-012**: The month navigator MUST be one shared control used by the budgets page and
  the transactions page, with identical behavior, wording, and accessibility metadata.
- **FR-013**: The month navigator MUST remain usable with keyboard-only navigation, with
  assistive technology, and on narrow screens where the header stacks the title, the
  navigator, and the page actions instead of crowding each other.
- **FR-014**: While the movements are loading, the navigator's actions MUST be disabled so
  a user cannot queue conflicting month changes.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: History reads MUST stay scoped to the signed-in user; the period is an
  additional filter and MUST NOT widen access to other users' movements. Period inputs
  MUST keep the existing format validation and MUST reject an inverted range.
- **SQR-002**: Backend automated coverage MUST include period-scoped totals, boundary
  dates, exclusion of pending and removed movements, zero-value months, and the
  unchanged all-time behavior without a period. Frontend coverage MUST include the shared
  navigator unit tests, filter-row layout and criterion behavior, view-level month
  defaults and clearing, plus an end-to-end journey that navigates months on the
  transactions page and verifies list, address, and totals together.
- **SQR-003**: No new environment secret, upload, or data-handling constraint applies;
  this feature reads existing financial data and introduces no new stored field.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): `GET /api/v1/financial-history` keeps its shape and
   gains period-scoped `meta.totals` for requests that carry `from` and `to`. The existing
   filter object drives both the listed movements and the totals so the two can never
   disagree. Authorization, validation, and pagination rules are unchanged.
2. **Frontend** (`../zunera-frontend`): The transactions filter bar places a shared month
   navigator beside the search and filter controls; the transactions view owns the
   selected month, sends the inclusive period with the other criteria, keeps the address
   in sync, and preserves the month when criteria are cleared. The budgets page adopts the
   same shared navigator without behavior change. Requirements depend on the backend
   period-scoped totals contract above.

### Key Entities *(include if feature involves data)*

- **Selected month period**: the calendar month currently displayed, expressed as an
  inclusive first-day and last-day range used for both listing and totals.
- **History totals**: realized income, expense, and net result for the requested period,
  derived from effective, non-removed income and expense movements; transfers and
  recognized credit-card expenses stay outside this aggregate.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user reaches any month's movements in two interactions or fewer from the
  transactions page.
- **SC-002**: 100% of months navigated show list entries and totals restricted to that
  month, including movements dated on the first and last day.
- **SC-003**: Reloading or sharing the transactions address restores the same month,
  criteria, and totals in 100% of attempts.
- **SC-004**: The budgets month control keeps its existing behavior with zero reported
  regressions in its automated checks.
- **SC-005**: The filter row stays free of overlap and remains keyboard operable at the
  smallest supported screen width.
- **SC-006**: A later month switch never leaves totals from the previously displayed
  month on screen.

## Assumptions

- The transactions page previously opened unfiltered over all history; defaulting to the
  current business month is an accepted, intentional behavior change.
- "Current month" follows the existing business time zone used by budgets.
- Only the period narrows totals; type, status, account, category, and search criteria keep
  their existing effect on the list without changing the totals.
- Recognized credit-card expenses keep their current behavior of appearing in the list
  without contributing to the realized income/expense cards.
- The removed-transactions screen, dashboard period presets, and existing budgets month
  behavior are out of scope.
- The feature reuses existing list, period, and pagination capabilities; no new stored
  field, migration, or third-party dependency is required.
