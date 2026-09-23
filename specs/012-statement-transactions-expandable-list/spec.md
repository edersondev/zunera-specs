# Feature Specification: Expandable Statement Transactions

**Feature Branch**: `012-statement-transactions-expandable-list`
**Backend Branch**: `012-statement-transactions-expandable-list` (`../zunera-backend`)
**Frontend Branch**: `012-statement-transactions-expandable-list` (`../zunera-frontend`)
**Created**: 2026-09-23
**Status**: Ready for implementation
**Input**: Statement transaction list improvement following Credit Cards Spec 010.

## User Scenarios & Testing

### User Story 1 - Scan and inspect statement purchases (Priority: P1)

As a card owner, I can scan compact statement entries and expand one purchase installment to inspect its existing details without leaving the statement.

**Why this priority**: Identifying a charge is the main task on the statement.

**Independent Test**: Open a statement with multiple installments, inspect their descriptions, dates, amounts and statuses, then expand and collapse them using pointer and keyboard.

**Acceptance Scenarios**:

1. **Given** a statement with multiple installments, **when** it opens, **then** every installment appears once with its original purchase description, date, sequence, amount and available status.
2. **Given** one expanded installment, **when** another is opened, **then** the first closes and the new details appear immediately below their header.
3. **Given** an installment with a credit adjustment, **when** details open, **then** original installment amount, adjustment and recognized amount remain distinct and exact.
4. **Given** no installments, **when** the statement opens, **then** the existing compact empty state appears.

### User Story 2 - Correct or refund from statement (Priority: P2)

As a card owner, I can use existing eligible purchase actions from an expanded statement entry and see the refreshed statement afterward.

**Why this priority**: Users should not have to return to card details to act on a statement purchase.

**Independent Test**: Open an eligible installment, correct or refund its purchase through the current dialog, and inspect refreshed statement values.

**Acceptance Scenarios**:

1. **Given** a directly editable purchase, **when** its installment expands, **then** Correct and Refund actions appear and open the existing dialogs.
2. **Given** a purchase in a closed statement, **when** its installment expands, **then** Correct is unavailable while Refund remains subject to its existing server rules.
3. **Given** a successful action, **when** the statement refreshes, **then** changed or removed installments and financial totals reflect the authoritative response without stale expansion.

### Edge Cases

- Long merchant names and large amounts stay readable at 320px without horizontal scrolling.
- Missing category or icon produces no invented label or icon.
- A purchase lookup or action error keeps the statement intact and surfaces existing error feedback.
- Light, dark and reduced-motion settings preserve readable focus and controls.

## Requirements

### Functional Requirements

- **FR-001**: Display each existing statement installment once in a compact, responsive list.
- **FR-002**: Show original purchase description, purchase date, installment sequence, unchanged installment amount, and available recognition status in collapsed state.
- **FR-003**: Show category name/icon, card identity, statement period, original purchase amount and adjustment details only when available.
- **FR-004**: Permit one expanded item at a time through accessible pointer and keyboard controls; clear expansion when that installment disappears.
- **FR-005**: Offer existing Correct only when directly editable and existing Refund for an owned purchase; retain current dialogs and validation.
- **FR-006**: Preserve separate payment and credit-event presentation, all financial calculations, and the statement empty state.

### Security and Quality Requirements

- **SQR-001**: Existing authenticated, owner-scoped statement and purchase endpoints remain protected; new response information is read-only and comes from the owned purchase.
- **SQR-002**: Cover response shape and unchanged amounts in backend tests, interaction and action wiring in frontend tests, and critical statement journeys in Playwright.
- **SQR-003**: No new secrets, uploads, packages, database changes, or financial mutations.

### Delivery Scope

1. **Backend** (`../zunera-backend`): Add only read-only purchase identity and display/action eligibility fields to statement installments; test response and preserve authorization and amount semantics.
2. **Frontend** (`../zunera-frontend`): Render expandable installments, reuse existing purchase actions and dialogs, refresh statement after mutations, and validate responsive themes and accessibility after backend contract passes.

### Key Entities

- **Statement**: Existing billing period, card identity, installments and authoritative financial totals.
- **Installment**: Existing purchase-linked line item with amount, credit adjustment, recognition status and stable ID.
- **Purchase**: Existing description, category, original amount, action eligibility and credit history.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Every statement installment appears exactly once with its unchanged amount and description.
- **SC-002**: A keyboard user can expand, inspect and collapse any item; no more than one item is expanded at a time.
- **SC-003**: At 320px, the list and details have no horizontal page overflow in either theme.
- **SC-004**: Correct and Refund complete through existing flows without changing financial values except through the authorized action.

## Assumptions

- This is a follow-up to Credit Cards Spec 010; the prompt's Spec 009 reference predates current repository numbering.
- Statement rows represent installments; refunds, corrections and payments remain financially separate in their existing sections.
