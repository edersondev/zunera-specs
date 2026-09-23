# Feature Specification: Transactions Activity Redesign

**Feature Branch**: `013-transactions-activity-redesign`
**Backend Branch**: Not created (frontend-only; existing contracts unchanged)
**Frontend Branch**: `013-transactions-activity-redesign` (`../zunera-frontend`)
**Created**: 2026-09-23
**Status**: Ready for implementation
**Input**: Redesign Spec 004 Transactions page as a monthly, date-grouped, expandable activity list.

## User Scenarios & Testing

### User Story 1 - Browse monthly activity (Priority: P1)

Users see activity grouped by date and can navigate months without losing financial meaning.

**Independent Test**: Open Transactions, change month across a year boundary, and inspect groups and amounts.

**Acceptance Scenarios**:

1. **Given** the Transactions page opens without a date query, **when** loading completes, **then** the current São Paulo business month controls the displayed activity.
2. **Given** December is selected, **when** the user advances, **then** January of the next year loads and pagination resets.
3. **Given** a custom date range, **when** a month arrow is selected, **then** the range visibly changes to the selected full month.
4. **Given** more results are loaded, **when** a page boundary bisects a date, **then** all entries for that date remain in one visible group.

### User Story 2 - Inspect and act on a movement (Priority: P2)

Users scan compact entries, open one entry at a time, and use only supported actions.

**Independent Test**: Expand income, expense, transfer, and card entry; inspect details and invoke existing actions.

**Acceptance Scenarios**:

1. **Given** an entry, **when** its header is activated by pointer or keyboard, **then** details appear below it and the expanded state is announced.
2. **Given** an open entry, **when** another is opened, **then** the first closes.
3. **Given** an action is selected, **when** it opens its existing flow, **then** expansion is not toggled and permissions/confirmation remain unchanged.
4. **Given** a filtered-out or removed open entry, **when** the list refreshes, **then** no stale details remain open.

### User Story 3 - Understand period results and filters (Priority: P3)

Users see reliable realized totals for the active complete date range and understand when totals cannot represent additional filters.

**Independent Test**: Navigate months and custom ranges, apply non-date filters, and compare summary visibility with activity.

**Acceptance Scenarios**:

1. **Given** a complete date range and no non-date filters, **when** it changes, **then** realized income, expense, and result update for that range.
2. **Given** search, type, status, account, or category filter, **when** applied, **then** non-comparable summary cards disappear while activity remains filtered.
3. **Given** no matching movements, **when** loading completes, **then** the empty message distinguishes no history, empty period, and filtered results where data permits.

### Edge Cases

- Empty or open-ended dates cannot produce a comparable period summary.
- Long descriptions and large amounts must not overlap at 320px width.
- Recurring projected entries and recognized credit-card expenses retain their distinct dates and action availability.
- Failed summary load must not replace accurate activity with stale totals.

## Requirements

### Functional Requirements

- **FR-001**: Display a reusable month navigator in the header, with correct year transitions and current-month default.
- **FR-002**: Preserve custom date range, search, type, category, account, status, filter chips, URL state, and load-more pagination.
- **FR-003**: Group existing mixed history entries by their existing movement date without changing server order or page size.
- **FR-004**: Render compact, responsive, expandable rows with one stable-ID expansion at a time and accessible keyboard interaction.
- **FR-005**: Display only existing fields and actions, with distinct income, expense, transfer, recurring, and card-expense presentation.
- **FR-006**: Show authoritative realized period totals only when comparable with active filters; never derive them from paginated rows.
- **FR-007**: Preserve creation, editing, status, deletion, transfer, recurrence, and credit-card financial behavior.

### Security and Quality Requirements

- **SQR-001**: Existing backend authorization and input validation stay unchanged; no new endpoint or write action.
- **SQR-002**: Unit and Playwright coverage must verify changed UI, date/filter coordination, accessibility, and financial presentation.
- **SQR-003**: No new secrets, upload paths, dependencies, or persistence changes.

### Delivery Scope

1. **Backend** (`../zunera-backend`): No changes. Existing financial-history and dashboard-summary contracts are already validated and consumed as-is.
2. **Frontend** (`../zunera-frontend`): Redesign Transactions view and feature components; reuse existing APIs, stores, dialogs, utilities, and theme tokens.

### Key Entities

- **History entry**: Existing mixed movement with kind, stable ID, date, amount, status, and type-specific relations.
- **Active period**: Existing `from`/`to` filter bounds; full month or explicit custom range.
- **Realized summary**: Existing date-bounded dashboard values for income, expense, and result.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Every loaded entry appears exactly once in a date group; no date ordering changes.
- **SC-002**: Month navigation, including December–January, refreshes activity and summary and resets to first page.
- **SC-003**: All supported entry actions remain reachable by keyboard; at most one row is expanded.
- **SC-004**: Page remains readable at 320px and 200% zoom, in light and dark themes.

## Assumptions

- Default period is current month in `America/Sao_Paulo`.
- A complete custom range takes precedence until a month arrow is chosen.
- Summary covers realized ordinary transactions, excluding pending entries, transfers, and card-recognized entries per existing backend calculation.
- Category icons are unavailable in history payload; use already-loaded category catalog where possible and a generic icon otherwise.
