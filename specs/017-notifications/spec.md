# Feature Specification: Notifications

**Feature Branch**: `017-notifications`  
**Backend Branch**: `017-notifications` (`../zunera-backend`)  
**Frontend Branch**: `017-notifications` (`../zunera-frontend`)  
**Created**: 2026-09-29  
**Status**: Ready for implementation  
**Input**: User description: "Create centralized, in-app financial notifications as an awareness and navigation layer over Zunera's authoritative domains."

The request calls this feature “Spec 012”; that number already belongs to Statement Transactions Expandable List in this repository. The next available directory is `017-notifications`. Numbering does not change requested scope.

## Clarifications

### Session 2026-09-29

- Q: Can users disable Critical notifications? → A: Yes. Every notification category is optional, including Critical alerts.
- Q: What happens when one change jumps a budget from below 80% to over 100%? → A: Emit only the highest current stage, Budget exceeded; never backfill lower stages from the same jump.
- Q: Should budget alerts cover category plans, the monthly total, or both? → A: Category plans only; no monthly-total threshold alert.
- Q: How long should read or resolved notifications remain in the center? → A: 90 days after creation or resolution, whichever is later; unresolved actions remain while their source condition persists.
- Q: If an unpaid statement first becomes eligible two days before due, should it receive the approaching reminder? → A: Yes. Send one approaching reminder on first eligibility 1–3 local calendar days before due.

### Session 2026-09-29

- Q: Should a failed recurring card occurrence generate a review notification? → A: Yes, when Spec 014 presents it as reviewable; reuse the occurrence's existing review notification without a duplicate.
- Q: What happens to budget threshold alerts when their calendar month ends? → A: Resolve them at month end; keep 90-day history and generate no new alerts for corrections to closed months.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Find and Understand Notifications (Priority: P1)

As a signed-in user, I want a global unread indicator and central history so I can spot important financial events without checking each feature.

**Why this priority**: Discovery and understandable state make every domain alert useful.

**Independent Test**: Present several notification types to one user. Verify global count, ordering, meaning, navigation, read controls, and unchanged financial state.

**Acceptance Scenarios**:

1. **Given** zero, one, several, or more than 99 unread items, **When** the header appears, **Then** notification access remains available and indicator shows no count, `1`, the exact count through `99`, or `99+`, respectively, with accessible meaning.
2. **Given** mixed history, **When** the center opens, **Then** newest items appear first and each shows meaning, time, read state, and whether action remains pending.
3. **Given** an unread actionable item, **When** the user opens it, **Then** it becomes read and opens authorized context, while underlying financial condition stays unchanged.
4. **Given** several unread items, **When** the user marks all as read, **Then** all currently accessible unread items become read, header count updates, and unresolved actions remain unresolved.
5. **Given** no history, **When** the center opens, **Then** a clear non-error empty state appears.

---

### User Story 2 - Act on Card Statement Deadlines (Priority: P1)

As a cardholder, I want timely unpaid-statement reminders that reflect the card's current obligation, so I can inspect it before or after its due date.

**Why this priority**: Due dates have direct financial impact.

**Independent Test**: Move a statement with outstanding balance through three days before due, due today, overdue, and paid. Check one alert per stage and accurate action state.

**Acceptance Scenarios**:

1. **Given** an eligible closed statement with outstanding balance one to three calendar days before due, **When** it first qualifies within that window, **Then** one attention reminder identifies masked card, statement period, due date, and outstanding amount.
2. **Given** it remains outstanding on due date, **When** due-today status arrives, **Then** one due-today item appears; earlier reminder stays historical but no longer presents as current action.
3. **Given** Credit Cards regards it as overdue with outstanding balance, **When** that state arrives, **Then** one critical overdue item appears and older stages cease to be active.
4. **Given** statement is fully paid before a stage is evaluated, **When** evaluation occurs, **Then** no unpaid reminder for that stage appears.
5. **Given** prior reminders, **When** statement becomes fully paid, **Then** related actionable reminders resolve without Notifications changing statement state.
6. **Given** a statement first becomes eligible two days before due, **When** its source state is evaluated, **Then** one approaching reminder appears. If it first qualifies on due date, only due-today stage appears.

---

### User Story 3 - Review Due Recurrences (Priority: P1)

As a user with recurring activity, I want notice when a generated occurrence needs review, so I can confirm or correct it in its source feature.

**Why this priority**: Pending occurrences are easy to miss.

**Independent Test**: Create one pending ordinary occurrence and one expected card purchase. Check each prompts review once, then resolve both through source features.

**Acceptance Scenarios**:

1. **Given** an ordinary generated transaction is pending and needs confirmation, **When** it is due, **Then** one attention item links to its review context.
2. **Given** a card occurrence is expected, awaits over-limit approval, or has failed and is reviewable under Spec 014, **When** review is needed, **Then** one attention item describes its actual state and links to the right action, without claiming issuer charge.
3. **Given** an occurrence is confirmed, made effective, dismissed, or canceled, **When** its source state changes, **Then** its item resolves even if never opened.
4. **Given** a rule is paused or ended while an earlier occurrence still needs review, **When** notifications are evaluated, **Then** that occurrence remains actionable under its own state.
5. **Given** a notified card occurrence changes from Expected to Failed, **When** it remains reviewable, **Then** its existing item updates to the failure/retry context and remains actionable without a second item.

---

### User Story 4 - Notice Budget Thresholds (Priority: P2)

As a user with monthly category plans, I want one notice per meaningful realized-spending threshold so I can adjust spending.

**Why this priority**: Existing budget status defines useful thresholds.

**Independent Test**: Move one plan through below 80%, 80%, 100%, above 100%, below 80%, and back above 80%; then start another month.

**Acceptance Scenarios**:

1. **Given** a category plan is below 80%, **When** Budgets classifies realized utilization as Approaching, **Then** one attention item appears for that plan and month.
2. **Given** realized utilization becomes exactly 100%, **When** Budgets classifies it as Reached, **Then** one distinct limit-reached item appears.
3. **Given** realized utilization exceeds 100%, **When** Budgets classifies it as Exceeded, **Then** one critical item appears with authoritative excess; earlier stages become historical. If one change jumped from below 80% to exceeded, no approaching or reached item is created for that jump.
4. **Given** utilization falls and crosses the same threshold again in one month, **When** the crossing recurs, **Then** no equivalent duplicate appears. Another month may receive its own alert.
5. **Given** only projected spending crosses a threshold, **When** realized utilization stays below it, **Then** no threshold item appears.
6. **Given** category plans together make the monthly total cross a threshold, **When** category alerts are evaluated, **Then** only qualifying category plans can notify; no monthly-total threshold item is created.
7. **Given** a budget month ends, **When** the next business month begins, **Then** its unresolved threshold items resolve and leave Requires action. A later correction to that ended month creates no new threshold item.

---

### User Story 5 - Celebrate a Goal Milestone (Priority: P2)

As a goal owner, I want to know when allocated progress reaches my target without Zunera silently completing the goal.

**Why this priority**: Goal attainment is meaningful, though less urgent than a deadline.

**Independent Test**: Fund a goal to target, drop below it, then reach the same target again. Verify one milestone and no automatic completion.

**Acceptance Scenarios**:

1. **Given** an active goal is below target, **When** authoritative allocation or accepted target change makes it reach or exceed target, **Then** one success item identifies goal and target and links to details.
2. **Given** progress falls below and later reaches the same target, **When** it rises again, **Then** no second equivalent milestone appears.
3. **Given** target changes to a distinct amount, **When** the goal first reaches that target, **Then** a new milestone may appear for the new target state.
4. **Given** a milestone appears, **When** it is read, **Then** the goal's status and allocation remain unchanged.

---

### User Story 6 - Choose Relevant Categories (Priority: P2)

As a user, I want simple preferences so future optional notifications match my interests without erasing history.

**Why this priority**: Control limits noise while retaining useful defaults.

**Independent Test**: Disable a category, cause a qualifying event, re-enable it, then cause a new event. Only the new event creates an item.

**Acceptance Scenarios**:

1. **Given** a new user, **When** preferences first appear, **Then** Credit Cards, Recurring Transactions including card recurrences, Budgets, and Financial Goals are enabled.
2. **Given** Budgets is disabled, **When** a new budget threshold occurs, **Then** no new Budget item appears; prior Budget history remains accessible.
3. **Given** Budgets is re-enabled, **When** a new event first qualifies, **Then** it may notify; events first qualified while disabled are not backfilled.
4. **Given** Credit Cards is disabled, **When** a statement becomes overdue, **Then** notification is suppressed while source statement remains available.

### Edge Cases

- Repeated evaluation, retry, duplicate source event, concurrent evaluation, and refresh produce at most one equivalent item per user and business context.
- Deleted, archived, or unavailable source loses pending-action affordance; historical item remains safe if still authorized. Lost authorization hides text, count, and destination entirely.
- Reading pending item leaves it pending. Source action elsewhere resolves it and marks it read. Canceling a recurring occurrence resolves alert; pausing a rule alone does not.
- A reviewable card occurrence failing after an expected or over-limit state retains one review item; a failed occurrence that later records or is dismissed resolves that item. Internal failure codes never appear in user-facing text.
- Payment before reminder creates no item. Payment afterward resolves it. Partial payment keeps current stage actionable with current outstanding amount.
- Budget fall and recross in same month does not repeat item; new month is distinct. Removed plan ends active attention.
- At the next local business-month boundary, prior-month budget threshold attention resolves. Historical spending corrections update Budgets but create no new Notifications alert for the ended month.
- If a notified statement or budget stage becomes current again after resolution, its existing retained item may show action pending again while remaining read; no new equivalent item appears. An expired equivalent event is not reissued.
- Goal reach, fall, and regain of same target does not repeat milestone. Archived goal cannot create new milestone.
- Disabling preferences preserves history. Re-enabling does not backfill first-qualified events from disabled interval.
- Empty history differs from empty filtered view. Large history remains navigable without loading everything at once.
- Timezone changes affect displayed local time and future business-day checks, not identity of items already emitted.
- Money, card identity, and destinations never reveal another user's data. User-authored labels display safely.

## Requirements *(mandatory)*

### Functional Requirements

#### Notification Center and meaning

- **FR-001**: Signed-in users MUST have global access to a center containing only currently authorized items. Items MUST sort by creation time descending, with stable tie ordering. Unresolved actionable items MUST be identifiable without jumping ahead of newer items.
- **FR-002**: Each item MUST convey type, title, short summary, event or creation date/time, read state, semantic level, originating domain, related context where applicable, pending/resolved state, and meaningful destination where available. Internal identifiers MUST NOT appear in user text.
- **FR-003**: Semantic levels MUST be Information, Attention, Success, and Critical based on user impact. Ordinary expenses, negative amounts, and routine operations MUST NOT become alerts solely because of amount or sign.
- **FR-004**: New items MUST begin unread. Opening an item MUST mark it read, then open its currently authorized destination where one exists. Individual and mark-all-as-read controls MUST exist. Mark-all MUST affect all currently accessible unread items, including outside visible history, but MUST NOT resolve source conditions.
- **FR-005**: Header indicator MUST count accessible unread items, actionable or informational, never total history. It MUST show no count at zero, `1` for one, exact `2`–`99`, and `99+` above that, with accessible meaning. It MUST reflect reads, resolution, retention, and authorization.
- **FR-006**: Center MUST offer All, Unread, and Requires action views. Requires action MUST include unresolved actionable items whether read or unread. Each view MUST retain newest-first order and distinguish no history from no filter matches.
- **FR-007**: Read and resolved MUST remain separate: reading MUST NOT resolve; resolution MUST remove item from Requires action and mark it read. A resolved item MUST remain understandable in history. Budget attention counts as actionable awareness, without requiring a mutation.
- **FR-008**: Destination MUST identify relevant statement, occurrence, category plan/month, or goal. Current source availability and authorization MUST be respected. If unavailable, show safe explanation or broader authorized context, never broken navigation or leaked details.

#### Initial notification matrix and transitions

| Business event | First qualifying state | Level | Unique business context |
| --- | --- | --- | --- |
| Statement approaching due | First eligibility within one to three local calendar days before due; eligible closed statement has positive outstanding amount | Attention | User + statement + approaching stage |
| Statement due today | Due date; same eligibility | Attention | User + statement + due-today stage |
| Statement overdue | Credit Cards considers statement overdue with positive outstanding amount | Critical | User + statement + overdue stage |
| Ordinary recurrence requires review | Generated occurrence pending and requiring confirmation | Attention | User + occurrence + review need |
| Card recurrence requires review | Expected, awaiting-over-limit, or Failed occurrence that Spec 014 marks reviewable | Attention | User + occurrence + review need; changed reason updates context, not identity |
| Budget approaching | In current budget month, Budgets reports realized category-plan utilization at least 80% and below 100% | Attention | User + category plan + calendar month + approaching stage |
| Budget limit reached | In current budget month, Budgets reports realized utilization exactly 100% | Attention | Same scope + reached stage |
| Budget exceeded | In current budget month, Budgets reports realized utilization above 100% | Critical | Same scope + exceeded stage |
| Goal target reached | Active goal first reaches/exceeds current target by authoritative allocation/progress | Success | User + goal + target state |

- **FR-009**: Only matrix events MUST notify initially. Ordinary and card recurrence review share one semantic type with source-specific detail. Routine transaction creation/edit, transfers, successful statement payment, automatic generation success, goal contribution, report generation, and budget edits alone MUST NOT create items.
- **FR-010**: Statement stages MUST be separate historical items. One approaching item MUST be created when an eligible statement first qualifies within one to three local calendar days before due; first eligibility on due date or later MUST produce only its current stage. At a later stage, earlier ones MUST cease to present current action; only latest applicable stage is actionable. Missed earlier stages MUST NOT be generated retrospectively. Fully paid or zero-outstanding statement MUST NOT receive unpaid reminders. If a payment reversal makes a previously notified statement stage current again, its retained item MUST become actionable again without becoming unread or creating an equivalent new item; an expired equivalent event MUST NOT be reissued. Credit Cards state and obligation are authoritative.
- **FR-011**: Statement text MUST identify masked card, statement period, due date, and current outstanding amount where available; no full card number. Partial payment MUST retain current-stage action with current outstanding amount. Full payment or effective credit adjustment MUST resolve all related stages.
- **FR-012**: Ordinary review MUST follow Spec 006's pending generated occurrence and ordinary transaction confirmation. Card review MUST follow Spec 014's Expected, Awaiting over-limit, and reviewable Failed occurrence states. A transition among those states MUST update one occurrence review item without creating another. Text MUST distinguish expected, approval, and failed/retry context; it MUST NOT claim charged/posted or expose internal failure codes. Rule pause/end MUST NOT resolve earlier due actionable occurrences; source confirmation, effectiveness, dismissal, cancellation, or equivalent resolution MUST resolve alerts.
- **FR-013**: Budget alerts MUST concern expense-category plans in the current business calendar month only; monthly-total utilization and corrections to ended months MUST NOT create threshold notifications. Status and realized utilization MUST come from Budgets, including card installment recognition. Expected/projected values MUST NOT trigger alerts. Initial approach threshold MUST match fixed 80%. Reached and Exceeded MUST each notify at most once per plan and period. If one source change crosses multiple unnotified thresholds, only the highest current stage MUST notify; skipped lower stages MUST NOT be backfilled later in that period, including if utilization falls. If plan drops to lower state, higher-stage item MUST no longer claim current action; recross in same period MUST NOT duplicate it. On recross, an existing retained item MUST become actionable again while remaining read; an expired equivalent event MUST NOT be reissued. Removing plan or reaching the next business-month boundary MUST resolve its active attention; the item remains in history for the normal retention period.
- **FR-014**: Goal eligibility MUST use Financial Goals' authoritative allocation and target. Reaching target MUST NOT complete goal. At most one milestone MUST be emitted per goal and target state, even after fall and regain. A distinct accepted target MAY create a new milestone if first reached while active. Archived goals MUST NOT create milestones.
- **FR-015**: Every qualifying business event MUST produce at most one equivalent item for same authorized user and context despite retries, repeated/concurrent evaluation, or duplicate delivery. Type, source entity, period/target, and stage define equivalence; wording does not.
- **FR-016**: When source condition ends, unresolved item MUST stop claiming pending action. Resolution MUST differ from read. Historical event meaning/time MUST remain; current amount/status shown as current context MUST match source or be clearly labeled event-time.

#### Preferences, history, integrity

- **FR-017**: Preferences MUST provide four categories: Credit Cards, Recurring Transactions (ordinary and card), Budgets, Financial Goals. All MUST default on. Users MUST be able to disable any category, including its Critical alerts; none is mandatory initially. Disabling Credit Cards MUST suppress new overdue-statement notifications as well as its other types without hiding the source statement. Changes MUST affect only future first-qualified events. Existing history remains; re-enabling MUST NOT backfill first-qualified events from disabled interval. A later distinct stage MAY qualify.
- **FR-018**: Resolved items and other nonactionable items, whether read or unread, MUST remain accessible for 90 calendar days after the later of creation or resolution, then leave the center. Unresolved actionable items MUST remain accessible while their source condition persists, regardless of age or read state. Notification history is not permanent financial audit history. Manual deletion and user-controlled retention are outside scope.
- **FR-019**: Views and actions MUST enforce current user/workspace boundaries from Authentication and source domains. If access is lost, item text, count, and destination MUST be hidden immediately; if regained within retention, access MAY resume. No stale destination or user label may leak another user's details.
- **FR-020**: Creation, reading, resolution, preferences, and retention MUST NOT change financial entities, balances, obligations, budget utilization, goal allocation, or source lifecycle. Source feature alone performs actions. Notifications MUST consume authoritative state and MUST NOT build parallel financial calculations.
- **FR-021**: “In 3 days,” “today,” and “overdue” MUST follow application's financial business date and user locale/timezone conventions with calendar-day semantics. Timezone change MUST NOT duplicate prior business event. Monetary values MUST follow existing currency and precision rules.
- **FR-022**: Presentation MUST follow Design Foundation, app shell, navigation, and components guidance: Teal and Slate/Neutral identity, semantic colors, Light/Dark/System themes, responsive layout, existing typography and spacing. Importance, read, and action state MUST be conveyed beyond color. Controls MUST support keyboard, visible focus, contrast, accessible count/status. Non-critical background items MUST NOT cause disruptive automatic announcements.
- **FR-023**: Initial delivery MUST be in-app only, while business meaning MAY support future channels. Email, SMS, WhatsApp, browser/mobile push, marketing, AI warnings, investment/bank alerts, custom reminders, shared notifications, sounds, rules builder, and external monitoring are out of scope.
- **FR-024**: Qualifying source change SHOULD become visible within five minutes under normal operation; date conditions SHOULD reflect within five minutes after business-day boundary. For this target, normal operation means the application, database, and notification evaluator are healthy and running, with no deliberately injected outage. Measure from accepted source commit or business-day boundary to the first authorized in-app response containing the item. Temporary failure MUST NOT create duplicates on recovery. This specifies timing, not scheduling technology.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: User-facing notification and preference actions MUST validate signed-in identity, authorized scope, source ownership, and allowed state change. Invalid/unauthorized actions MUST leave notification, preference, and financial state unchanged without leaking details.
- **SQR-002**: Automated acceptance coverage MUST include every matrix row; transitions; repeated/concurrent evaluation; read versus resolution; preference changes; retention; timezone boundaries; removed/unauthorized sources; financial integrity; and critical navigation/accessibility journeys. Plan-phase contracts MUST cover changed request/response behavior.
- **SQR-003**: User-provided labels MUST display safely. No uploads, new secrets, payment credentials, or third-party delivery are required. Data handling MUST follow authorization and retention, including exclusion of unauthorized items from count and views.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define and enforce eventual application contract, authorization, validation, lifecycle, source-state integration, deduplication, preferences, retention, and tests. Contract shape and technical design belong to Plan.
2. **Frontend** (`../zunera-frontend`): After backend contract and tests, deliver accessible global access, Notification Center, destination handling, preferences, and journey coverage against contract. Screen/component/state design belongs to Plan.

### Key Entities

- **Notification**: User-specific record of one meaningful source event, with semantic level, event context/time, read state, and optional action/resolution state. Neither a financial transaction nor financial audit ledger.
- **Originating context**: Authorized source feature and statement, occurrence, monthly category plan, or goal, including business stage, period, or target needed to identify one event.
- **Notification preference**: User choice for one of four categories, affecting future first-qualified events only.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In moderated tests, at least 90% of participants find center, identify latest actionable item, and reach relevant context in under 60 seconds without help.
- **SC-002**: In acceptance scenarios, 100% of repeated/concurrent evaluations of same business event yield one equivalent notification per eligible user/context.
- **SC-003**: In acceptance scenarios, 100% of source resolutions remove pending-action presentation without notification actions changing financial values.
- **SC-004**: In acceptance scenarios, 100% of items and unread counts exclude other users' or newly unauthorized context.
- **SC-005**: With a healthy application, database, and notification evaluator, at least 95% of qualifying events appear in an authorized in-app response within five minutes of accepted source commit or applicable business-day boundary. Benchmark at least 20 qualifying events spanning source changes and date-boundary conditions; record each start and visibility time separately. Deliberately injected outage/recovery runs are evaluated for eventual delivery and deduplication, outside this latency sample.
- **SC-006**: With 10,000 retained items for one user, at least 95% of center openings/view changes show first recent items within three seconds; older history remains navigable.
- **SC-007**: At least 90% of test participants distinguish “read but still requires action” from “resolved” without opening source feature.

## Assumptions

- Existing specs are authoritative by feature despite numbering mismatch in request: Authentication `001`, Financial Accounts `002`, Categories `003`, Transactions `004`, Transfers `005`, Recurring Transactions `006`, Dashboard `008`, Budgets `009`, Credit Cards `010`, Recurring Credit Card Purchases `014`, Financial Goals `015`, Financial Reports `016`. Their current rules prevail over example wording.
- Statement due/overdue and outstanding values come from Credit Cards, including partial payments and credits. A zero-balance open cycle is not an unpaid statement reminder.
- Budget alerts use category-plan status for named calendar month; whole-budget summary creates no duplicate. Goal milestone concerns active allocated target, not explicit completion.
- “Requires action” for Budget means awareness/review, not mandatory mutation. It ends when stage is no longer current. Goal milestone is informational.
- Card approach notice uses first eligibility one to three local calendar days before due. Four categories enabled and 90-day history after creation/resolution are confirmed product choices.
