# Feature Specification: Financial Goals

**Feature Branch**: `015-financial-goals`\
**Backend Branch**: `015-financial-goals` (`../zunera-backend`)\
**Frontend Branch**: `015-financial-goals` (`../zunera-frontend`)\
**Created**: 2026-09-25\
**Status**: Ready for planning\
**Input**: User description: "Create Financial Goals so users can designate existing money for objectives, manage allocations and withdrawals, follow progress and history, and preserve established financial calculations."

## Clarifications

### Session 2026-09-25

- Q: Can users allocate money to a goal without an associated financial account? → A: Yes. Allow user-declared allocations, label account backing as unverified, and never add them to assets.
- Q: What happens when a new linked allocation exceeds the account's current balance after other goal allocations? → A: Reject it; if later real account activity causes a shortfall, show the shortage and block further linked additions without rewriting goal history.
- Q: Must a completed goal be reopened before its allocation can be withdrawn? → A: Yes. Explicitly reopen it to active status first; only then allow withdrawal.
- Q: Can a funded active goal be archived before its allocation is withdrawn? → A: No. Require an explicit full withdrawal before archival; preserve that withdrawal in goal activity.
- Q: Can an archived goal be restored to active status? → A: Yes. Restore it with complete history intact and zero allocated money.
- Q: May a target-funded goal be completed while its linked account has a shortfall? → A: No. Block completion until the linked account's shortfall is resolved; account-free goals remain eligible with an unverified-backing label.
- Q: May a target-funded goal be completed while its linked account is inactive or unavailable? → A: No. Require the owner to link an eligible active account or unlink the goal first; unlinked completion remains labelled unverified.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create a Goal and See Progress (Priority: P1)

As a signed-in user, I want to name an objective, set a target, optionally identify money already saved, and see progress without changing my financial accounts.

**Why this priority**: A goal and its accurate progress are the feature's essential value.

**Independent Test**: Create a R$ 30.000,00 goal with R$ 3.000,00 initially allocated from an account holding R$ 10.000,00; verify 10% progress, R$ 27.000,00 remaining, and unchanged account balance and total assets.

**Acceptance Scenarios**:

1. **Given** an active account has R$ 10.000,00 and no linked allocations, **when** its owner creates a R$ 30.000,00 goal linked to it with R$ 3.000,00 initially allocated, **then** the goal shows R$ 3.000,00, account balance remains R$ 10.000,00, unallocated account money is R$ 7.000,00, and no income, expense, or transfer appears.
2. **Given** a user has no account to link, **when** they create a valid goal without initial allocation or target date, **then** it is active at R$ 0,00 and 0% with no invented date or account.
3. **Given** a goal has R$ 8.500,00 allocated toward R$ 30.000,00, **when** viewed, **then** progress is about 28.33% and R$ 21.500,00 remains.
4. **Given** a goal has no linked account, **when** its owner allocates R$ 500,00, **then** the goal shows R$ 500,00 designated with an unverified account-backing label, while total assets and account balances remain unchanged.

---

### User Story 2 - Allocate and Withdraw Designated Money (Priority: P1)

As a user, I want to designate more existing money for a goal and release it when my priorities change, with a history I can understand.

**Why this priority**: Allocation and withdrawal keep goals useful over time while protecting accounting integrity.

**Independent Test**: Add R$ 500,00 to a goal at R$ 5.000,00, then withdraw R$ 200,00; verify R$ 5.300,00 allocated, two dated events, and no change in account balance or cash flow.

**Acceptance Scenarios**:

1. **Given** an active goal has R$ 5.000,00, **when** its owner allocates R$ 500,00, **then** it has R$ 5.500,00 and the addition appears once in goal activity.
2. **Given** the goal has R$ 5.500,00, **when** its owner withdraws R$ 200,00, **then** it has R$ 5.300,00, the release appears once, and no income or account credit is created.
3. **Given** a linked account holds R$ 20.000,00 and two goals designate R$ 10.000,00 and R$ 5.000,00, **when** another allocation exceeds R$ 5.000,00, **then** it is rejected without partial change and the account still displays R$ 5.000,00 unallocated.

---

### User Story 3 - Manage Goal Lifecycle (Priority: P2)

As a user, I want to acknowledge achievement, reopen a goal if needed, and archive an abandoned goal without losing its history.

**Why this priority**: Clear states prevent completed or abandoned goals from confusing current plans.

**Independent Test**: Reach a target, complete and reopen the goal, withdraw the allocation, archive and restore it; verify state, history, and unchanged actual finances at each step.

**Acceptance Scenarios**:

1. **Given** an active goal reaches its target and is either unlinked or linked to an active account without shortfall, **when** viewed, **then** it remains active and completion is available; **when** its owner completes it, **then** it is completed and retains its allocation.
2. **Given** a completed goal, **when** its owner tries to allocate, withdraw, edit its target, or archive it, **then** the action is rejected with guidance to reopen; reopening preserves allocation and history.
3. **Given** an active goal has positive allocation, **when** its owner tries to archive it, **then** archival is blocked until the allocation is explicitly withdrawn; once zero, archival preserves its history and changes no account balance.
4. **Given** an archived goal has zero allocation, **when** its owner restores it, **then** it becomes active with zero allocation, retains all earlier activity, and may receive new allocations under normal eligibility rules.
5. **Given** an active linked goal has reached its target but its account's combined goal allocations exceed the actual balance, **when** its owner attempts completion, **then** completion is blocked with the shortfall explained and the goal stays active; once the shortfall is resolved, completion is available. An account-free goal with target-level allocation may be completed with its unverified-backing label retained.
6. **Given** a target-funded goal is linked to an inactive or unavailable account, **when** its owner attempts completion, **then** completion is blocked until the goal is linked to an eligible active account or unlinked; unlinking preserves allocation and completion retains the unverified-backing label.

---

### User Story 4 - Understand Account Association and Shortfalls (Priority: P2)

As a user, I want to know where designated money is held and when a real account balance no longer covers its linked goals.

**Why this priority**: Linked goals must never imply money exists twice or remains available after spending.

**Independent Test**: Link two goals to one account, record a real expense from that account, and verify an explicit shortage with unchanged goal history and truthful account balance.

**Acceptance Scenarios**:

1. **Given** a real transaction lowers an account below its combined goal allocations, **when** its owner views the goals, **then** the exact shortfall and negative unallocated amount are labelled as requiring attention; no allocation is silently reduced.
2. **Given** a goal's linked account becomes inactive, **when** viewed, **then** its former account is labelled inactive and additions are blocked until reassociation or unlinking.
3. **Given** a funded goal changes its associated account, **when** the new account has sufficient unallocated money, **then** no transfer or balance change occurs, and prior activity retains its original association; otherwise the change is rejected.

---

### User Story 5 - Plan for a Target Date (Priority: P2)

As a user with a dated objective, I want to see time remaining and a simple monthly contribution suggestion so I can plan for myself.

**Why this priority**: Dates add useful context without implying a forecast or automatic movement.

**Independent Test**: Set a future date, verify the rounded suggestion from remaining amount and contribution periods, then pass the date and verify that overdue wording replaces the suggestion.

**Acceptance Scenarios**:

1. **Given** an active goal has R$ 12.000,00 remaining across 12 applicable monthly periods, **when** viewed, **then** it suggests R$ 1.000,00 per month as guidance and creates no recurring activity.
2. **Given** an underfunded goal is due today or overdue, **when** viewed, **then** its timing is labelled and no monthly contribution recommendation appears.
3. **Given** a goal has no date or has reached its target, **when** viewed, **then** no monthly contribution is suggested.

---

### User Story 6 - Review Goals Without Distorting Financial Views (Priority: P2)

As a user, I want an overview and a small dashboard summary while account balances, budgets, and cash-flow figures keep their established meanings.

**Why this priority**: Goals should be visible without becoming a second asset or spending ledger.

**Independent Test**: Change several goals and compare overview, dashboard, accounts, budgets, and credit-card figures before and after; only goal-specific values change.

**Acceptance Scenarios**:

1. **Given** active, completed, and archived goals, **when** the overview opens, **then** its target, allocated, and remaining totals cover active goals only; completed and archived goals remain distinguishable and accessible.
2. **Given** active goals, **when** the dashboard opens, **then** a compact selection shows progress and access to all goals; balances, realized and expected cash flow, financial result, and budget utilization are unaffected by goal operations.
3. **Given** a user buys something for a goal, **when** they record the real expense through Transactions or Credit Cards, **then** it follows that feature's recognition rules; goal allocation changes only through an explicit goal withdrawal.

### Edge Cases

- Reject whitespace-only or over-200-character name input, zero or negative targets, zero or negative allocation/withdrawal, malformed or over-precision BRL amounts, and amounts beyond the supported limit. Failed actions change neither history nor financial state.
- Reject withdrawal above current allocation, including concurrent withdrawals whose combined value exceeds it. Repeated submission of one action cannot double-count it.
- Reaching or exceeding a target never caps allocation or silently changes status. Reducing a target below allocation preserves money and shows excess.
- A past target date cannot be newly set. A once-future date may become due or overdue without changing status or creating an entry.
- Real transactions may later reduce a linked account below its allocations or below zero. Preserve true account balance and allocation history, show the shortfall, and block further linked additions until sufficient capacity returns.
- Unlinked goals can hold user-declared allocations, but their backing is unverified. They never create extra assets or a verified account balance.
- Inactive or unavailable linked accounts retain historical identity where possible. An active goal may withdraw, unlink, or reassociate, but cannot add while linked to that account.
- Completed goals cannot receive allocations or withdrawals until reopened. Archived goals are read-only except for restoration and are excluded from active planning totals until restored.
- A linked goal cannot be newly completed during an account shortfall. If real activity creates a shortfall after completion, the completed status and history remain intact, and the shortfall is shown as requiring attention rather than silently reopening the goal.
- A linked goal cannot be newly completed while its account is inactive or unavailable. If that account becomes inactive or unavailable after completion, the completed status remains and the account warning stays visible.
- Empty states, light/dark/system themes, narrow screens, keyboard use, assistive technology, and high zoom must keep values and actions understandable without color alone.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Only an authenticated owner MUST create, view, edit, complete, reopen, archive, restore, allocate to, or withdraw from a goal. Another user's goals, accounts, and goal history MUST be inaccessible.
- **FR-002**: A goal MUST have a name whose input is at most 200 characters, is trimmed before storage, and contains at least one non-whitespace character after trimming; whitespace-only input MUST be rejected. It MUST also have a positive BRL target, an owner, and active status at creation. Initial allocated amount, target date, eligible financial account, and description MUST be optional. Omitted initial allocation means R$ 0,00. An owner MAY use the same name for more than one goal; names are labels, not unique identities.
- **FR-003**: Monetary input and calculations MUST preserve cent precision. Positive targets and operation amounts MUST be R$ 0,01 through R$ 9.999.999.999,99; initial allocation MAY be zero. Invalid, malformed, over-precision, or out-of-range values MUST be rejected with actionable feedback.
- **FR-004**: A goal MUST be a designation of existing resources, never a financial account, income, expense, transfer, transaction, credit-card asset, or additional asset balance.
- **FR-005**: Initial allocation MUST be traceable goal activity and follow the same account eligibility and capacity rules as later allocation. Zero initial allocation MUST NOT fabricate a positive allocation event.
- **FR-006**: Each accepted allocation MUST add its exact amount once and record type, amount, business date/time, and account association at the time, if any.
- **FR-007**: Each accepted withdrawal MUST subtract its exact amount once and preserve corresponding activity. It MUST NOT exceed current allocation or make it negative.
- **FR-008**: Creating, allocating, withdrawing, editing, completing, reopening, or archiving a goal MUST NOT itself create or alter income, expenses, transfers, financial account balances, credit-card obligations, budgets, recurring activity, or independent transaction history.
- **FR-009**: Current allocation MUST equal accepted initial and later allocations minus accepted withdrawals. Displayed progress and summaries MUST use that authoritative amount after retries or simultaneous actions.
- **FR-010**: A goal MUST be active, completed, or archived. Reaching the target MUST leave it active until the owner explicitly completes it. Completion MUST require allocation at least equal to target and, for a linked goal, an active available associated account with no current shortfall. An owner MAY unlink or validly reassociate a goal before completing it. An account-free goal MAY be completed at target allocation with its unverified-backing label retained. A later account shortfall, archival, or unavailability MUST NOT silently reopen or change a completed goal's status; the account warning remains visible.
- **FR-011**: A completed goal MUST preserve allocation and history and MUST reject allocation, withdrawal, target/date/account edits, and archival until its owner explicitly reopens it. Reopening MUST make it active without altering money or prior activity; withdrawal is then available under active-goal rules.
- **FR-012**: An active goal MUST be archivable only when allocation is zero. The owner MUST explicitly withdraw the full remaining allocation in a separate goal action before archiving; that withdrawal MUST remain in activity history. An archived goal MUST retain its details and history, be read-only except for restoration, and be excluded from active totals and normal action choices. Its owner MUST be able to restore it to active status with zero allocation and prior history intact. Permanent deletion is outside scope.
- **FR-013**: Each accepted status change, including restoration, MUST be traceable by type and date/time. Archival and restoration MUST NOT delete or alter independent financial transactions or account balances.
- **FR-014**: The owner MUST be able to edit an active goal's name, positive target, optional target date that is today or later, optional eligible account, and optional description. Completed and archived goal metadata MUST remain read-only until the completed goal is reopened or the archived goal is restored. Metadata edits MUST preserve all historical goal activity and current allocation.
- **FR-015**: Changing target MUST immediately recalculate progress and remaining amount. Lowering it below allocation MUST be allowed, label excess, and cause no automatic completion or withdrawal.
- **FR-016**: A newly linked account MUST be an active, owned financial account that can hold money. Credit cards, credit limits, other users' accounts, and inactive accounts MUST NOT be selectable.
- **FR-017**: A goal MAY have no linked account and MAY receive initial or later allocations while unlinked. Its allocated amount MUST be labelled user-declared and unverified as to account backing; it MUST NOT count as extra assets or verified account funds.
- **FR-018**: For an active linked account, unallocated amount MUST equal actual current balance minus current allocations of all active and completed goals linked to it. Actual balance, designated amount, and unallocated amount MUST be distinguished.
- **FR-019**: At the time it is accepted, a linked allocation MUST be rejected if it would make total linked allocations exceed that account's nonnegative current actual balance. The same check MUST apply to initial allocation, concurrent additions, and changing a funded goal's association, accounting for release from the old account. Rejection MUST leave all goals unchanged.
- **FR-020**: If later real activity or correction makes linked allocations exceed actual balance, retain true balance and goal history, show negative unallocated amount and exact shortfall as an attention state, and block further linked allocation until resolved. No compensating transaction or automatic allocation reduction may occur.
- **FR-021**: Changing an association MUST NOT create a transfer, imply money moved, or rewrite prior activity's account context. Unlinking MUST free the old account's designated amount without changing actual balance or goal allocation.
- **FR-022**: If a linked account becomes inactive or unavailable, its identity MUST remain understandable in goal and activity history. New allocations MUST be blocked while linked to it. Withdrawal, unlinking, and valid reassociation MUST remain available for an active goal.
- **FR-023**: Multiple goals MAY link to one account. Capacity and unallocated figures MUST count active and completed goal allocations, because completion does not release designated money.
- **FR-024**: Goal details and active-goal presentation MUST show name, status, current allocation, target, percentage, remaining amount, optional date/account, and available actions. Progress MUST have numeric and text equivalents; color alone MUST NOT convey status.
- **FR-025**: Percentage MUST equal allocation divided by target times 100, showing at least one decimal where rounding to a whole number would mislead. Remaining MUST equal the greater of target minus allocation and zero. Above target, show full allocation, percentage above 100%, and excess separately; a visual indicator MAY stop at 100%.
- **FR-026**: With a target date, details MUST show that date and remaining calendar time or due/overdue status according to the user's financial business date. No achievement prediction follows from date or current progress alone.
- **FR-027**: For an active underfunded goal with a future date, suggested monthly contribution MUST equal remaining amount divided by the count of calendar-month contribution opportunities from current month through target month, inclusive, rounded up to the next cent. Partial current and target months each count once. It MUST be labelled guidance, not a guarantee or transaction.
- **FR-028**: No suggestion MUST appear without a date, at/above target, or when due today/overdue. Suggestions MUST NOT create recurring transactions, transfers, or allocations.
- **FR-029**: Goal details MUST show ordered goal-specific history with type, amount when applicable, date/time, and account at the time when applicable. Goal activity MUST be distinguishable from the financial transaction ledger.
- **FR-030**: The overview MUST show active and completed goals with access to archived history and active-goal totals for target, allocated, and remaining amounts. Totals MUST sum each active goal's own target, allocation, and nonnegative remaining amount; the allocated total MUST identify its unverified account-free portion. Overfunding MUST appear separately, not silently offset another goal's shortage.
- **FR-031**: The overview MUST identify overdue underfunded goals, account shortfalls, and inactive/unavailable linked accounts as attention states. Empty, loading, and unavailable states MUST be clear and MUST NOT present unknown values as zero; an empty overview MUST explain how to create a goal.
- **FR-032**: When active goals exist, Dashboard MUST show up to three active goals, ordered by overdue target date first, then nearest future target date, then goals without dates, with a stable name order for ties. Each shown goal MUST include progress, remaining amount, relevant target date, and access to all goals. When none exist, Dashboard MUST omit the section. Goal activity MUST NOT change Dashboard balances, realized/expected income or expenses, financial result, recurring activity, or card figures.
- **FR-033**: Goal operations MUST NOT consume a category budget or create budget income. Actual spending for a goal MUST use existing Transactions or Credit Cards rules; actual movement between accounts MUST use Transfers. Such real activity MUST NOT automatically change goal allocation in this version.
- **FR-034**: Existing rules for effective versus expected values, transfer exclusion, card purchase and statement-payment recognition, and recurring card purchase recognition MUST remain authoritative. Goal activity MUST NOT duplicate those amounts.
- **FR-035**: Views and actions MUST follow Zunera Design Foundation's Teal and Slate/Neutral identity, established semantic financial colors, typography and spacing, Light/Dark/System themes, responsive behavior, accessible contrast, visible focus, and text equivalents for progress and status.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Every action MUST validate ownership, account eligibility, goal/account status, dates, and monetary values when performed. Failed validation MUST leave goal and financial state unchanged and explain the reason without exposing another user's information.
- **SQR-002**: Acceptance coverage MUST include creation, allocation, withdrawal, capacity conflicts, concurrent/repeated actions, lifecycle, account changes, summaries, financial integrity, and critical user journeys.
- **SQR-003**: Names and notes MUST be handled as user-provided content without unsafe display. This feature handles no uploads or new financial credentials; it MUST retain history needed to explain past allocations and status changes.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Enforce ownership, validation, lifecycle, allocation and capacity rules, history, and consistent financial calculations before user-facing integration relies on them.
2. **Frontend** (`../zunera-frontend`): Provide goal overview and details, defined actions and guidance, and compact Dashboard integration using established business rules and accessible design guidance.

### Key Entities

- **Financial Goal**: A user's objective with a name, target, current allocation, status, optional target date, optional financial-account association, and optional description. It is neither an account nor an asset.
- **Goal Activity**: A dated, traceable allocation, withdrawal, or lifecycle event belonging to a goal, with amount and account context where relevant. It is not itself a financial transaction.
- **Financial Account**: Existing place where a linked goal's money is said to be held. Its actual balance stays authoritative and separate from goal allocation.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In representative usability checks, at least 90% of users create a named goal with target and optional date/account/initial amount in under 3 minutes without help.
- **SC-002**: Across acceptance cases for initial/later allocation, withdrawal, target edit, overfunding, association change, and multiple goals on one account, 100% of displayed goal amounts, percentages, remaining values, and history match the rules to the cent.
- **SC-003**: Across acceptance cases for goal actions and real transaction/transfer activity, 100% of account balances, total assets, Dashboard income/expenses/result, and budget utilization follow their existing rules without duplicate goal amounts.
- **SC-004**: In usability checks, at least 90% of users identify an account shortfall, distinguish actual balance from designated and unallocated money, and find the next corrective action without help.
- **SC-005**: With up to 100 goals and 1,000 goal activities, users can open the overview and goal details with usable information within 2 seconds under normal service conditions.

## Assumptions

- Existing financial business-date and BRL precision conventions apply. A selected financial account belongs to the same user as its goal.
- Financial Accounts, Transactions, Transfers, and card statement payments define actual account balance, even if later activity makes previously designated funds unavailable. A goal shortfall is an attention state, never a rewritten transaction or allocation.
- Unlinked goals allow planning before an account is selected, but cannot verify where money is held. Their allocation is not an additional asset.
- Completion is an explicit acknowledgement after reaching target. Archival means the objective is no longer planned; allocation must be explicitly released first so no hidden reservation remains. Restoration permits a new planning period without erasing the old one.
- Dependencies are Financial Accounts (spec 002), Categories (003), Transactions (004), Transfers (005), Recurring Transactions (006), Dashboard (008), Budgets (009), Credit Cards (010), and Recurring Credit Card Purchases (014). The user's legacy labels for Dashboard, Budgets, Credit Cards, and recurring card purchases differ from the repository's current numbering; current files govern cross-references.
- Initial scope excludes investment returns or interest projections, automatic investment, bank/Open Finance connections, automatic transfers, shared goals, templates marketplace, AI advice, links from individual expenses to goal consumption, complex forecasting, credit-limit funding, and automatically scheduled contributions.
