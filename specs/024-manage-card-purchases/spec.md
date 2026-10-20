# Feature Specification: Manage Credit Card Purchases

**Feature Branch**: `024-manage-card-purchases`
**Backend Branch**: `024-manage-card-purchases` (`../zunera-backend`)
**Frontend Branch**: `024-manage-card-purchases` (`../zunera-frontend`)
**Created**: 2026-10-20
**Status**: Ready for planning
**Input**: User description: "Move a purchase between statements; show a Statement select for a new purchase dated in a closed cycle; delete purchases."

## Clarifications

### Session 2026-10-20

- Q: May a purchase with linked refund or correction credits be permanently deleted? → A: No. Block Delete and direct the owner to existing correction actions.
- Q: After individual installment moves, what should a later Full schedule move do with custom placements? → A: Rebuild consecutive installments from the selected first statement, removing earlier gaps or shared statements.
- Q: May a One installment move reverse the billing order of installment numbers? → A: No. Statement dates must remain in installment order, but multiple installments may share a statement.
- Q: Which statement should be selected initially for a backdated purchase in a closed cycle? → A: None. Require the owner to choose a statement explicitly before saving.
- Q: How far ahead may a user choose a statement for a backdated purchase or move? → A: Any same-card billing cycle closing on or after the purchase date, including an empty future cycle.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Move a Purchase Between Statements (Priority: P1)

As a card owner, I can move a purchase's full installment schedule or one selected installment to another statement, so charges appear in the intended billing period.

**Why this priority**: Statement placement controls what I owe and when spending is recognized.

**Independent Test**: Move a two-installment purchase once as a full schedule and once as one installment; verify destination, unchanged amounts, source totals, and other installments.

**Acceptance Scenarios**:

1. **Given** an owned purchase, **When** I choose Move, **Then** I can choose Full schedule or One installment and see the current and proposed statement periods before confirming.
2. **Given** a full-schedule move, including one after individual installments were moved, **When** I select an eligible first statement and confirm, **Then** every installment keeps its amount and sequence and appears once in consecutive statements beginning with the selected statement; prior gaps or shared statements are removed, even if the first installment stays in its current statement.
3. **Given** a one-installment move, **When** I select an installment and different eligible destination statement, **Then** only that installment changes statement; every other installment keeps its statement, amount, and sequence, and installment numbers remain in statement-date order.
4. **Given** a source statement is closed or paid, **When** an eligible move succeeds, **Then** statement obligations and recognized spending reflect the new placement, existing payments stay recorded, any excess settlement appears as card credit, and the move's source and destination remain traceable.
5. **Given** a target belongs to another card, closes before purchase date, would reverse installment billing order, or would leave every installment in its current statement, **When** I try to move, **Then** no data changes and I see a specific reason.
6. **Given** a successful move, **When** affected views refresh, **Then** the purchase appears in its selected statement, source and destination totals agree, and a repeated confirmation does not move it again.
7. **Given** an eligible future billing cycle has no purchases yet, **When** I select it as a move destination, **Then** it is available and receives the selected installment or full schedule after confirmation.

---

### User Story 2 - Choose Statement for a Backdated New Purchase (Priority: P1)

As a card owner entering a purchase after its date's statement has closed, I can choose which eligible statement receives it instead of accepting only automatic historical placement.

**Why this priority**: Late entry can otherwise bill a purchase in an unintended cycle.

**Independent Test**: Enter a date in a closed cycle, choose a later same-card statement, save, and verify first installment placement. Enter an open-cycle date and verify normal automatic placement.

**Acceptance Scenarios**:

1. **Given** my chosen purchase date belongs to a statement already closed as of the current business date, **When** I enter that date, **Then** a labelled Statement select appears with no choice preselected and eligible same-card statements showing billing period, closing date, and due date; I must choose one before saving.
2. **Given** I select an eligible later statement, **When** I save, **Then** the first installment uses it and any later installments follow consecutive cycles; saved purchase and statement views agree.
3. **Given** my date belongs to a statement still open, including its inclusive closing date, **When** I enter it, **Then** the extra select is hidden and automatic date-based placement applies.
4. **Given** I change the date or card after choosing a statement, **When** the Statement select remains visible, **Then** the choice clears and I must choose again. A missing or stale submitted choice is rejected without creating a purchase.
5. **Given** an eligible future billing cycle has no purchases yet, **When** I choose it for a backdated purchase, **Then** the purchase can be saved into that cycle and later installments follow consecutive cycles.

---

### User Story 3 - Permanently Delete a Purchase (Priority: P2)

As a card owner, I can permanently delete a mistaken purchase so its charge and all installments disappear from card and spending records.

**Why this priority**: Mistaken entries should be removable without a replacement cancellation charge.

**Independent Test**: Confirm deletion of an owned purchase and verify its installments disappear, all affected totals are correct, and an existing statement payment remains recorded. Dismiss a second deletion and verify no change.

**Acceptance Scenarios**:

1. **Given** an owned purchase, **When** I choose Delete, **Then** I see a confirmation that names the purchase, states the number of installments, and warns that removal is permanent.
2. **Given** I confirm, **When** deletion completes, **Then** purchase and installments disappear permanently, affected financial totals update once, and no cancellation or refund credit is created.
3. **Given** I dismiss confirmation, **When** it closes, **Then** no financial data changes.
4. **Given** a deleted purchase was on a paid or partly paid statement, **When** deletion completes, **Then** payments and paying-account movements remain, statement obligation falls, and excess settlement is visible as card credit.
5. **Given** a purchase has any linked refund or correction credit, applied or not, **When** I try to delete it, **Then** deletion is blocked with no data changes and I am directed to existing correction actions.
6. **Given** a purchase came from a recurring occurrence, **When** I delete it, **Then** the recurrence rule and other occurrences remain unchanged, and that occurrence does not recreate its purchase automatically.
7. **Given** a completed deletion, **When** I revisit purchase and statement views or repeat the request, **Then** the purchase remains absent and no second balance change occurs.

### Edge Cases

- Purchase date equals closing date; that date remains in the open cycle until the next business day.
- Configured closing day exceeds days in a month, including February and leap years.
- Backdated purchase belongs to a cycle that has no stored statement yet.
- Eligible destination is an empty future cycle, including one beyond the next upcoming statement.
- Statement becomes ineligible after form choices load or between preview and confirmation.
- Full-schedule move spans year end, billing-day changes, both open and closed statements, or an installment schedule previously changed by individual moves.
- A stored statement with an old closing day and a newly calculated cycle share a calendar month; a purchase date after the stored statement closes still finds the new cycle.
- A purchase with move history is directly edited while its current installments remain open; descriptive edits keep its statement placement and audit, while allocation-changing edits fail without mutation.
- Single-installment move puts two installments on one statement or leaves a gap; original sequence labels remain visible and statement dates stay in sequence order.
- Move or delete changes a closed, partly paid, paid, or card-credit-adjusted statement.
- Repeated or simultaneous commands must not apply money changes twice.
- Archived or foreign card, purchase, statement, or category must not be exposed or selected.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Owners MUST be offered Full schedule and One installment modes when moving a purchase. The choice and affected statement periods MUST be clear before confirmation.
- **FR-002**: A Full schedule move MUST place the first installment in the selected destination and every later installment in its next consecutive billing cycle, even if earlier individual moves created gaps or shared statements. It MUST preserve sequence, each amount, exact total, purchase date, category, and card.
- **FR-003**: A One installment move MUST change only the chosen installment's statement. Other installments MUST retain their original statements, amounts, and sequence. Resulting statement closing dates MUST remain in nondecreasing installment-number order; multiple installments MAY share a statement or leave an empty cycle.
- **FR-004**: A move destination MUST belong to the purchase's card and close on or after the purchase date. A One installment move MUST change that installment's statement; a Full schedule move MUST change at least one installment's statement, but MAY keep the first installment in place while rebuilding the rest consecutively. Any eligible future billing cycle MUST be selectable even if it has no existing activity, subject to existing supported date limits. Full-schedule moves MUST also have valid subsequent cycles. Invalid, unavailable, foreign, or stale choices MUST cause no partial change.
- **FR-005**: The move preview MUST identify affected installments, source and proposed destination periods, closing dates, due dates, and resulting statement charges. Confirmation MUST be required.
- **FR-006**: The new-purchase Statement select MUST appear only when the date-derived statement has already closed by the current business date. It MUST start unselected and require an explicit choice before saving. It MUST let the owner select any same-card billing cycle closing on or after the purchase date, including empty future cycles within existing supported date limits, and show each choice's period, closing date, and due date.
- **FR-007**: With the select hidden because the date-derived statement remains open, existing date-based purchase allocation MUST apply. When the select is shown, the chosen eligible statement MUST receive the first installment and subsequent installments MUST use consecutive cycles; an omitted choice MUST be rejected.
- **FR-008**: Changing date or card MUST refresh statement choices and clear the previous choice whenever the select remains visible. Missing, invalid, or stale submitted choices MUST produce field-level feedback and no purchase.
- **FR-009**: Delete MUST require explicit confirmation describing permanent removal and the affected purchase and installment count. Dismissal MUST make no change.
- **FR-010**: Confirmed Delete MUST permanently remove the purchase and all installments from purchase and statement history. It MUST NOT create a cancellation or refund credit. Deletion MUST be blocked whenever any linked refund or correction credit exists, including unapplied credits, and direct the owner to existing correction actions.
- **FR-011**: Move and Delete MUST restate affected statement totals, statuses, outstanding balances, card credit, available credit, category spending, budgets, dashboard, financial history, and statement notifications exactly once. Neither action MUST create, erase, or repeat a paying-account cash movement.
- **FR-012**: Effective statement payments MUST remain recorded against their original statement and paying account after a move or deletion. Excess settlement MUST become visible card credit under existing application order; outstanding balance MUST never become negative.
- **FR-013**: Success MUST refresh purchase lists, affected statement views, and financial summaries. Failure MUST show actionable feedback and leave the previous financial state intact.
- **FR-014**: Repeated or overlapping confirmed move or delete requests MUST NOT duplicate financial changes. A request to delete an already deleted purchase MUST have no second money effect.
- **FR-015**: Deleting a purchase created from a recurring occurrence MUST preserve its rule and other occurrences and MUST NOT recreate the deleted purchase automatically.
- **FR-016**: A completed move MUST leave a readable history of the affected installment, original and destination statement, and time of change, including when either statement was closed.
- **FR-017**: When the existing open-statement correction rule permits a direct edit of a moved purchase, changes limited to category, description, or notes MUST preserve installment IDs, statement assignments, recognition dates, and move history. A direct edit that changes card, purchase date, total amount, or installment count after any move MUST return an actionable conflict with no partial change. Permanent Delete MUST remove the move history with the purchase.
- **FR-018**: The date-derived cycle MUST use a stored statement whose preserved period contains the purchase date, choosing the earliest closing date when stored periods overlap; otherwise it MUST use the card's current billing days. A stored statement in the same calendar month MUST NOT suppress a distinct cycle calculated from current billing days. Statement choices MUST accept an eligible stored historical closing date or the exact calculated closing date for an empty cycle, subject to the purchase-date and ownership rules.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Changed create, move, and delete actions MUST require authentication and owner-scoped authorization for card, purchase, installment, statement, category, and related financial records. Foreign records MUST not be disclosed.
- **SQR-002**: Dates, ownership, statement eligibility, purchase state, installment effects, and linked financial adjustments MUST be validated before mutation. Conflicts MUST cause no partial financial effect.
- **SQR-003**: Automated coverage MUST include changed request and response contracts, ownership, calendar boundaries, installment arithmetic, closed and paid statements, overlapping requests, derived financial views, and critical dialog-to-result journeys.
- **SQR-004**: No new secrets, uploads, issuer connection, or card credentials are involved. Existing private financial-data protections apply.
- **SQR-005**: Form choices, move and delete confirmation, errors, and resulting financial states MUST work with keyboard and assistive technology, narrow screens, 200% zoom, and Light, Dark, and System themes without relying on color alone.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define the protected contract for statement choices, purchase creation, both move modes, and permanent deletion; enforce validation and authorization; preserve payment and financial consistency; verify all changed financial outcomes before frontend delivery.
2. **Frontend** (`../zunera-frontend`): Add the conditional Statement select to the new purchase dialog and both move modes and Delete to purchase actions; show previews, confirmation, errors, and refreshed views using the established backend contract; cover critical journeys.

### Key Entities *(include if feature involves data)*

- **Credit Card Purchase**: An owner's dated, categorized card charge with amount, installment count, and optional recurring source.
- **Credit Card Installment**: One ordered part of a purchase with an amount and assigned statement; all parts sum to the purchase total.
- **Credit Card Statement**: A card's billing period with closing and due dates, charges, payments, credit adjustments, outstanding balance, and status; an eligible empty cycle can become a statement when selected.
- **Statement Payment and Card Credit**: Existing settlement and excess-credit records that remain financially accurate when a purchase changes or disappears.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In a scripted usability check with 20 card-feature users using fixture data (five each for Full schedule/mobile, One installment/mobile, Full schedule/desktop, and One installment/desktop), at least 19 complete their assigned move from the purchase list within 2 minutes, measured from opening Move to confirmation, without facilitator help.
- **SC-002**: In a deterministic 100-case matrix of distinct single- and multi-installment moves spanning month and year boundaries, every installment appears once in its selected statement and all amounts sum to the original purchase total. A separate moved-purchase correction check preserves every installment ID and move-history row after an allowed descriptive edit and rejects each allocation-changing field without changing either.
- **SC-003**: In a deterministic 100-case date matrix including dates before, on, and after closing day and old/new closing days within one month, backend allocation and the form agree: the Statement select appears exactly when the date-derived cycle is already closed, begins unselected, requires a choice, and the saved purchase uses that eligible statement. A specific changed-billing-day case proves that a January 10 stored statement does not suppress a January 25 calculated cycle for a January 15 purchase.
- **SC-004**: In a deterministic 100-case matrix of moves and deletions across open, closed, partly paid, and paid statements, charges, recognized spending, balances, payments, and card credit reconcile to the centavo with no duplicate cash movement.
- **SC-005**: In a deterministic 100-attempt matrix of repeated or overlapping move and delete requests, one confirmed operation changes financial totals at most once.
- **SC-006**: In the same 20-user scripted check as SC-001, at least 18 correctly name the installments affected by a move and recognize that Delete is permanent before confirming; after a confirmed action, they locate the changed statement or totals within 30 seconds without facilitator help.

## Assumptions

- Existing Credit Cards 010 rules remain authoritative except where this feature explicitly allows moving purchases across closed statements and permanently deleting them. The owner chose permanent deletion over the earlier cancellation-only rule.
- A date-derived statement is "already closed" only after its inclusive closing date passes in the app's business calendar. A purchase dated on today's closing day is still in an open cycle.
- Late entry into a closed cycle always requires an explicit Statement choice, including when the owner wants the date-derived statement.
- Statement reassignment changes billing allocation, not purchase date. A target cannot close before that purchase date.
- Eligible statement choice has no additional horizon or existing-activity requirement; an empty future cycle is valid within existing supported date limits.
- One installment may move without changing the rest while statement dates remain in installment-number order; it may share a statement with another installment or leave a gap. A later Full schedule move replaces those custom placements with consecutive cycles.
- Existing statement payments remain cash history after reassignment or deletion. Existing card-credit rules handle any excess settlement.
- A purchase with any linked refund or correction credit cannot be permanently deleted in this feature, regardless of application state; existing correction actions remain available.
- Card billing-day settings, issuer data, statement payment actions, and recurrence schedules do not change.
