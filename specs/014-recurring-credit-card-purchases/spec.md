# Feature Specification: Recurring Credit Card Purchases

**Feature Branch**: `014-recurring-credit-card-purchases`

**Backend Branch**: `014-recurring-credit-card-purchases` (`../zunera-backend`)

**Frontend Branch**: `014-recurring-credit-card-purchases` (`../zunera-frontend`)

**Created**: 2026-09-24

**Status**: Clarified after planning; ready for tasks

**Input**: Extend existing Recurring Transactions so eligible expense occurrences can become single-payment purchases on an owned credit card, while preserving account recurrence and credit-card accounting rules.

## Clarifications

### Session 2026-09-24

- Q: Which generation mode is selected by default for a new card recurrence? → A: Automatic purchase generation. Owners may explicitly choose confirmation-based generation.
- Q: May confirmation use a different actual amount for one occurrence? → A: Yes. The confirmed purchase uses that amount while the rule and later occurrences retain the scheduled amount.
- Q: What if automatic generation requires over-limit approval? → A: Keep the occurrence awaiting explicit owner approval; create no purchase until the owner approves under existing card rules.
- Q: May the owner enter a different actual purchase date when confirming one occurrence? → A: Yes. Default to the scheduled date, but allow a valid owner-entered purchase date for that occurrence; statement assignment and recognition follow the chosen date.
- Q: How may an owner complete an expected occurrence whose card or category became unavailable? → A: Allow an explicit eligible card or category replacement for that occurrence only; preserve its original association and source for audit without changing the rule or other dates.
- Q: Does pausing or ending a rule cancel occurrences already due but unconfirmed? → A: No. Already due occurrences remain confirmable or dismissible under current purchase rules; pause or end only prevents future generation.
- Q: Where does a late automatic purchase go if its scheduled date's statement already closed? → A: Use the original scheduled purchase date and statement; restate that statement, its recognition month, obligation, and available credit without duplicating prior payments.
- Q: May an existing recurrence change between Financial Account and Credit Card destinations? → A: No. Destination type is fixed at creation; the owner may change the selected account or card within that type, or end the old rule and create a new rule to switch types.
- Q: May a confirmed card occurrence use an actual purchase date after today's São Paulo business date? → A: No. The actual purchase date may differ from the scheduled date but must be on or before the current São Paulo business date; future-dated confirmation is rejected without a purchase.
- Q: May an owner dismiss an automatic occurrence awaiting over-limit approval? → A: Yes. Dismissal declines that one charge without creating a purchase, permanently reserves its scheduled date against regeneration, and leaves future cycles governed by the rule.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Schedule a Credit Card Expense (Priority: P1)

As an authenticated user, I want to choose a credit card as the destination of a recurring expense, so that subscriptions and memberships do not require repeated manual purchase entry.

**Why this priority**: A clear recurrence definition is the foundation for every subsequent occurrence.

**Independent Test**: Create a monthly R$ 150,00 gym recurrence on an active C6 Bank card ending in 3450, then view its card, category, amount, schedule, generation behavior, and next expected date in the existing recurrence experience.

**Acceptance Scenarios**:

1. **Given** an owner has an active card and available expense category, **When** they choose Credit Card, card, monthly frequency, and valid amount and dates, **Then** one owned expense rule is saved and identified as a credit-card recurrence.
2. **Given** Financial Account is selected, **When** an existing account recurrence is created or edited, **Then** its established account selection, pending transaction generation, and balance behavior remain unchanged.
3. **Given** an unavailable or foreign card, income classification, incompatible category, or invalid amount or dates, **When** the user saves, **Then** no rule or purchase is created and the invalid field is explained.
4. **Given** a card recurrence is viewed, **When** the owner reads its details, **Then** card identity and the difference between a new purchase each cycle and installments of one purchase are clear without relying on color.

---

### User Story 2 - Generate a Fixed Charge Automatically (Priority: P1)

As an authenticated user, I want a predictable due charge to become a purchase on my selected card, so that card obligations and statements reflect it without re-entry.

**Why this priority**: Reliable generation delivers the primary time saving while protecting financial totals.

**Independent Test**: Configure an automatic R$ 150,00 charge for the 12th, process an eligible due date repeatedly and concurrently, and verify one linked single-payment purchase, one statement allocation, and no ordinary account expense.

**Acceptance Scenarios**:

1. **Given** an active automatic card rule with an eligible due date and valid associations, **When** the date becomes due, **Then** at most one purchase for that rule and date is recorded with its card, description, category, amount, and occurrence date.
2. **Given** a purchase date is the card's statement closing date, **When** the purchase is recorded, **Then** it belongs to the statement closing that day; due date and financial recognition follow existing card rules.
3. **Given** generation is retried, overlaps, or resumes after downtime, **When** due dates are processed, **Then** each eligible active date produces at most one purchase and no duplicate obligation or expense.
4. **Given** a purchase is recorded, **When** balances and reports are viewed, **Then** available credit and statement obligation reflect it, financial-account balances do not change, and spending follows the statement's recognition period.
5. **Given** automatic generation would exceed available credit, **When** the date is processed, **Then** the occurrence waits for explicit owner approval with the resulting negative available credit explained, and no purchase or card obligation exists until approval.
6. **Given** an occurrence awaits over-limit approval, **When** the owner reviews current credit and explicitly approves under existing card rules, **Then** one purchase uses the original occurrence date and repeated approval cannot create another.
7. **Given** generation fails before recording a purchase, **When** the issue is resolved and processing retries, **Then** one purchase may be recorded for that original date without a partial or duplicate obligation.
8. **Given** processing resumes after the scheduled date's statement has closed, **When** an eligible automatic occurrence is recorded late, **Then** its purchase enters the original statement and closing-month expense, affected statement and credit totals are restated once, and prior effective payments are not repeated.
9. **Given** an automatic occurrence awaits over-limit approval, **When** the owner dismisses it, **Then** no purchase or card obligation is created, that date is never regenerated, and later cycles remain eligible.
10. **Given** several missed automatic dates, **When** a middle date fails to record, **Then** earlier committed purchases remain, that date is durably failed and retryable when possible, and later dates still process; if even its failure identity cannot be stored, processing stops without advancing past it.

---

### User Story 3 - Confirm an Expected Charge (Priority: P1)

As an authenticated user, I want to review a due card charge before recording it, so that a variable or uncertain subscription is not mistaken for an actual purchase.

**Why this priority**: A schedule is a forecast, not proof that an issuer charged the card.

**Independent Test**: Let a confirmation-based occurrence become due, verify it remains expected with no card obligation, confirm it once, and verify a single linked purchase and clear change in reporting.

**Acceptance Scenarios**:

1. **Given** a confirmation-based card rule reaches a due date, **When** the owner reviews it, **Then** the occurrence is labelled Expected and no purchase, statement line, used credit, realized expense, or account movement exists yet.
2. **Given** an eligible expected occurrence, **When** the owner confirms it, **Then** one single-payment purchase is recorded and linked to that occurrence; repeated confirmation cannot record another.
3. **Given** the scheduled amount is R$ 150,00 and the actual charge is R$ 165,00, **When** the owner confirms that one occurrence at R$ 165,00, **Then** its purchase uses R$ 165,00 while the rule and later occurrences retain R$ 150,00.
4. **Given** an occurrence is scheduled for the 12th and today is the 13th or later, **When** the owner enters the 13th as its actual purchase date and confirms, **Then** that one purchase uses the 13th for statement assignment and recognition while its recurrence source remains the 12th.
5. **Given** the owner declines an expected charge, **When** it is dismissed, **Then** no purchase is recorded for that occurrence, its disposition remains traceable, and future dates remain governed by the rule.
6. **Given** a purchase was recorded from confirmation, **When** it is reviewed, **Then** Zunera describes it as recorded in Zunera without claiming external issuer verification.
7. **Given** the owner enters a future actual purchase date, **When** they try to confirm an expected occurrence, **Then** the date is rejected and no purchase or statement line is created.
8. **Given** two confirmations for the same occurrence submit different actual values concurrently, **When** one attempt has claimed the occurrence, **Then** the other cannot overwrite its choices and receives a retryable conflict; the eventual purchase and recorded occurrence show the same winning amount, date, card, and category.
9. **Given** a confirmation worker stops after claiming an occurrence, **When** the owner retries after safe recovery, **Then** Zunera first checks whether a purchase was committed and records no second purchase or conflicting actual values.
10. **Given** a card rule has at least one expected, awaiting-over-limit, or failed occurrence, **When** the owner views the recurrence list, **Then** that rule is visibly marked for review; after its last such occurrence is recorded or dismissed, the mark disappears. Recorded and dismissed occurrences alone do not mark a rule.
11. **Given** a card rule has actionable occurrences, **When** its compact list item is viewed, **Then** a subtle warning accent, explicit review badge and count, and direct Review occurrence action distinguish occurrence attention from the rule's lifecycle state and signed amount.
12. **Given** multiple actionable occurrences, **When** the owner chooses Review occurrence from the list, **Then** the newest actionable occurrence opens in the existing review dialog; the expanded item previews its scheduled date, expected amount, and occurrence state using existing data.

---

### User Story 4 - Manage Future Rules and Historical Purchases (Priority: P2)

As an authenticated user, I want to edit, pause, resume, or end a card recurrence without rewriting past purchases, so that subscription changes leave statements auditable.

**Why this priority**: Recurrence management must not silently change recorded obligations.

**Independent Test**: Generate one purchase, change the rule's amount, category, card, and schedule, pause and resume it, then verify the original purchase and statement remain unchanged while only future eligible dates use the new definition.

**Acceptance Scenarios**:

1. **Given** a rule has generated purchases, **When** amount, category, card, charge day, frequency, or end date changes, **Then** existing purchases and presented occurrences retain their values; only future ungenerated dates use the edit.
2. **Given** an active rule is paused or permanently ended, **When** future schedule dates pass, **Then** no new purchase is generated; existing purchases, statements, and recognized spending remain.
3. **Given** a purchase needs correction, cancellation, or refund, **When** the owner changes that individual purchase under card rules, **Then** the template and other occurrences remain unchanged and no replacement purchase is generated for the same date.
4. **Given** a selected card or category becomes unavailable, **When** a new due date approaches, **Then** the rule pauses with an actionable reason; history remains readable and generation resumes only after an eligible replacement and explicit resume.
5. **Given** an owner tries to switch an existing rule from account to card or card to account, **When** they edit it, **Then** the change is rejected with guidance to end that rule and create another, while past occurrences and future scheduling of the unchanged rule remain intact.
6. **Given** an expected occurrence's card or category becomes unavailable, **When** the owner explicitly selects an eligible replacement for that occurrence and confirms it, **Then** one purchase uses the replacement, the original association remains auditable, and the rule and other occurrences remain unchanged.
7. **Given** a charge was due before its rule was paused or ended, **When** the owner later reviews that expected occurrence, **Then** they may confirm or dismiss it under current purchase rules without reviving future rule dates.
8. **Given** a card charge became due while processing was unavailable, **When** the owner edits, pauses, or ends the rule before catch-up, **Then** that already-due date is represented using the old definition before the mutation takes effect; if it cannot be represented, the rule mutation is rejected without changing the template or undoing earlier due purchases.

---

### User Story 5 - Understand Reports Without Double Counting (Priority: P2)

As an authenticated user, I want recurrence expectations, card purchases, and statement payments to have distinct meanings in Budgets and Dashboard, so that my financial picture remains trustworthy.

**Why this priority**: Repeated charges affect multiple views but must represent one obligation and one recognized expense.

**Independent Test**: Compare one unconfirmed occurrence, one recorded purchase in an Open statement, the same installment after closing, and a subsequent statement payment across Budget, Dashboard, card, and account views.

**Acceptance Scenarios**:

1. **Given** an unconfirmed due charge, **When** Dashboard upcoming activity is viewed, **Then** it appears once as an expected charge, not as a purchase or realized expense.
2. **Given** a generated purchase has an Open statement, **When** Budget and Dashboard are viewed, **Then** its single pending installment is expected in the statement-closing month and is not also counted as a recurrence expense.
3. **Given** that statement closes, **When** financial views refresh, **Then** the installment moves to realized spending once in the closing month; payment later settles obligation without new category or budget spending.
4. **Given** a future eligible card charge is within Dashboard's upcoming horizon, **When** the owner views upcoming activity, **Then** it appears once as expected and contributes nothing to realized totals or account balances.

### Edge Cases

- Monthly 29th, 30th, or 31st uses the last day of a shorter month and returns to its configured day when available; yearly 29 February follows the existing leap-year rule.
- A due date coincides with statement closing, crosses year end, or uses a card whose closing/due day is absent in that month.
- A rule starts in the past, processing resumes after downtime, a rule is paused across due dates, or its inclusive end date passes; existing recurrence eligibility and catch-up rules apply.
- A late generated purchase enters a statement already closed or paid; its original statement, recognized spending, obligation, and available credit restate without creating a second payment or account movement.
- Card or category is archived or otherwise unavailable before generation or before an expected occurrence is confirmed. Historical identities remain visible; invalid associations cannot create a purchase.
- A purchase would exceed available credit. Automatic generation leaves it awaiting explicit owner approval; available credit and approval warning are reassessed under existing card rules when the owner acts.
- An owner dismisses an automatic occurrence awaiting over-limit approval; the scheduled date remains reserved and no financial effect occurs.
- Generation fails before a purchase is recorded, after a partial attempt, during retry, or concurrently with confirmation, edit, pause, or cancellation.
- A backlog has several eligible dates and a middle date fails; independently committed earlier dates remain, later dates continue after a durably represented failure, and an unrepresentable failure stops the rule before cursor progress.
- An owner edits a card rule after a missed due date but before processing catches up; the missed date uses pre-edit values and a concurrent processor cannot let the edit skip or rewrite it.
- Competing confirmations choose different actual amount/date/card/category values, or a confirmation worker stops after saving an attempt; one purchase and its occurrence audit fields must agree after safe recovery.
- An occurrence is confirmed twice, dismissed then retried, or its purchase is corrected, cancelled, or refunded. Its rule/date identity remains reserved and auditable.
- A confirmation-based occurrence is confirmed with an actual purchase date after the current São Paulo business date; validation rejects the future date without changing occurrence disposition or financial values.
- A user attempts to switch destination type on an existing rule; the change is rejected. Changing the selected account or card within the same type affects only future unrepresented dates, while historical records retain their original destination and meaning.
- A card purchase is allocated to an Open statement, then billing days change; existing statement allocation remains fixed.
- A user attempts to see or act on another user's rule, card, occurrence, purchase, category, statement, or financial summary.
- All journeys work at narrow widths, 200% zoom, keyboard-only and assistive-technology use, and Light, Dark, and System themes.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Authenticated owners MUST manage card recurrences through the existing Recurring Transactions experience. No separate subscription management area is created.
- **FR-002**: Each recurrence MUST have exactly one payment destination identity: an owned financial account or an owned credit card. Destination type is fixed when the rule is created; changing from account to card or card to account requires ending the existing rule and creating another. Creation and same-type reassignment MUST use an active destination; later archival preserves historical association and blocks future generation until resolved. Existing account-based rules retain their destination and behavior without user migration.
- **FR-003**: Credit-card destination MUST be available only for expense recurrences with an available owned expense category. Income recurrences remain account-based. Invalid, archived, unavailable, or foreign associations MUST be rejected before a rule or purchase is recorded.
- **FR-004**: Card rules MUST retain existing description, positive BRL amount, category, optional note, weekly/monthly/yearly frequency, start date, optional inclusive end date, and active/paused/ended lifecycle rules. Existing amount, text, date, and calendar bounds apply.
- **FR-005**: Card rules MUST support automatic and confirmation-based generation as an explicit rule choice. New card rules default to automatic purchase generation; owners MAY choose confirmation-based generation before saving. Existing account rules keep their pending-transaction behavior without a required new choice.
- **FR-006**: Automatic generation MUST use the rule amount. In confirmation-based generation, the owner MAY enter a different positive actual amount for one occurrence, subject to existing BRL purchase amount bounds and validation. The occurrence MUST retain its scheduled amount for comparison, while its purchase uses the confirmed amount; changing one occurrence MUST NOT edit the rule or later occurrences.
- **FR-007**: A card recurrence definition is a schedule only; it MUST NOT itself create an account movement, card obligation, statement line, budget realization, or realized Dashboard expense.
- **FR-008**: Existing recurrence eligibility applies: a past start does not backfill dates before rule creation; due dates missed while active after eligibility begins are processed once after downtime when associations remain valid; paused dates are skipped; inclusive end date and ended state prevent later occurrences.
- **FR-009**: An active automatic card rule MUST produce one single-payment card purchase for each eligible due date when all purchase rules pass. It MUST NOT also produce an ordinary account transaction or claim that the issuer verified or charged the card.
- **FR-010**: A confirmation-based card rule MUST present one expected occurrence on its eligible due date without recording a purchase. Confirmation records one single-payment purchase; dismissal records no purchase for that date and prevents automatic regeneration of that date. Both decisions remain traceable and do not alter other dates.
- **FR-011**: Generated purchases MUST preserve source rule and scheduled occurrence date, card, description, category, amount, and note where applicable. Automatic purchases use the occurrence date as purchase date. Confirmation defaults purchase date to the occurrence date but MUST allow the owner to enter a different valid actual purchase date for that occurrence without changing the rule or its scheduled identity. The confirmed actual date MUST be on or before the current São Paulo business date, in addition to existing card date bounds; a future date MUST be rejected without financial effect. Purchases appear in ordinary card and statement activity with visible recurrence origin.
- **FR-012**: A card occurrence MUST create at most one purchase for a rule and scheduled date across retries, concurrent processing, repeated confirmation, or editing. A failed attempt before recording a purchase remains retryable without duplicate effect; a corrected, cancelled, or refunded purchase never reopens that date for generation. For catch-up across multiple due dates, a later failure MUST NOT roll back purchases or represented occurrences already committed for earlier dates. A recoverable failed date MUST remain represented and retryable while later eligible dates continue; if even its failure identity cannot be persisted, processing MUST stop for that rule without advancing past the unrepresented date.
- **FR-013**: Card recurrence purchases MUST have exactly one payment. Each cycle is a new purchase, never an installment of a multi-cycle purchase. Configuring a multi-installment recurring purchase is outside initial scope.
- **FR-014**: Purchase eligibility, card ownership, category eligibility, available credit, over-limit confirmation, corrections, cancellation, refunds, and card credit MUST follow existing Credit Cards rules. If automatic generation needs over-limit confirmation, it MUST leave the occurrence awaiting explicit owner approval without recording a purchase or changing card obligation. The owner MAY dismiss that occurrence instead, permanently reserving its date without financial effect or changing future cycles. If the owner approves, current available credit MUST be reassessed and any still-required over-limit approval MUST show the resulting negative available credit before one purchase may be recorded. No automatic action may claim issuer authorization.
- **FR-015**: Each purchase MUST follow existing statement assignment from purchase date and billing cycle. A purchase on closing date belongs to the statement closing that day; due date, status, later billing-day changes, and closing-month recognition follow existing card rules. A late automatic purchase MUST use its original scheduled date; a confirmed purchase MUST use the owner's chosen valid purchase date. When either date belongs to a statement already closed or paid, affected statement totals, recognized spending in its original closing month, obligation, and available credit MUST restate exactly once without repeating effective payments or changing account balances.
- **FR-016**: A purchase MUST affect card obligation, used/available credit, and statement balances exactly as a manually entered purchase. It MUST NOT reduce an account balance merely because it was recorded.
- **FR-017**: Its single installment MUST contribute Expected spending while its statement is Open and become Realized once in the calendar month of closing. Rule and unconfirmed occurrence MUST NOT contribute a second expense; statement payment settles obligation without extra category, Budget, or Dashboard spending.
- **FR-018**: An eligible future card schedule date or due unconfirmed card occurrence within Dashboard's upcoming horizon MUST appear once as a scheduled expectation, clearly separate from purchases and realized totals. Under existing Budget rules it MUST NOT enter Expected spending before a purchase installment exists. After purchase creation, the installment is the sole Expected/Realized source in its statement-closing month.
- **FR-019**: Dashboard account balances, realized income/expenses, result, recent activity, upcoming activity, card obligations, due statements, and available credit MUST retain existing definitions, including transfer exclusion from income and expenses. A recorded card purchase MUST appear once in applicable recent activity; the same rule/date MUST not appear simultaneously as separate expected recurrence and recorded purchase in one activity view.
- **FR-020**: Account recurrences MUST continue to create pending ordinary transactions and follow established pending/effective balance, Budget, Dashboard, lifecycle, and origin behavior. No historical account occurrence is converted to card purchase.
- **FR-021**: An owner MAY edit an active card rule's amount, category, selected card, generation behavior, schedule, frequency, start date, or optional end date only for future dates not represented by an occurrence or purchase. An account rule MAY change its selected account under existing rules. Destination type MUST NOT change after creation. Before applying an owner-initiated card rule edit, pause, or end, Zunera MUST process or durably represent every eligible date already due while the rule was active under its pre-change definition, including dates missed during downtime. A represented `failed` or `awaiting_over_limit` date retains that definition and may be acted on later. If a due date cannot even be durably represented, the mutation MUST be rejected with a retryable explanation and leave the rule unchanged; independently committed earlier due purchases remain valid. Purchases and presented expected occurrences retain snapshots. Edits MUST not recreate a represented date under the same rule.
- **FR-022**: Pausing, resuming, auto-pausing for unavailable associations, and permanently ending a card rule MUST follow existing recurrence lifecycle rules. Pause or end blocks future generation but never removes historical purchases, changes statement balances, or reverses spending. An occurrence already due before pause or end MUST remain confirmable or dismissible if current purchase rules permit; acting on it MUST NOT resume or revive the rule.
- **FR-023**: If an active rule's destination or category is missing or unavailable, the rule MUST pause and create no future purchases until the owner provides an eligible rule association and resumes it; an already ended rule remains ended. A presented expected occurrence cannot be confirmed against an unavailable association. Its owner MAY explicitly replace that occurrence's card or category with an eligible owned association before confirmation, preserving the original association and scheduled source for audit; that one-occurrence change MUST NOT alter the rule, other occurrences, or its lifecycle state. It remains identifiable until confirmed or dismissed, without silent reassignment.
- **FR-024**: Editing, correcting, cancelling, or refunding one generated purchase MUST use individual card-purchase rules and restate affected statements, spending, and credit exactly once. It MUST NOT modify source rule or another occurrence.
- **FR-025**: Recurrence list and detail MUST identify destination type, card using existing name/institution/last-four conventions, generation behavior, schedule, lifecycle state, next expected date, and source links without color-only cues. Existing recurrence filters MUST remain available and allow users to distinguish account and card rules. Contextual guidance replaces the card-unavailable message and explains recurrence versus installments.
- **FR-026**: Creation, confirmation, dismissal, editing, lifecycle changes, blocked associations, over-limit decisions, and failures MUST provide clear, actionable, privacy-safe feedback. Failure MUST NOT leave a partial purchase, duplicate obligation, or ambiguous occurrence disposition.
- **FR-027**: Expected, awaiting over-limit approval, dismissed, failed, and recorded describe only the occurrence workflow. Expected means a due charge awaits confirmation; awaiting over-limit approval means automatic recording was blocked pending the owner's explicit approve-or-dismiss decision; dismissed means owner declined it and its date cannot regenerate; failed means recording did not complete and may be retried when valid; recorded means one linked purchase exists. None is a new purchase, installment, statement, or issuer-verification financial state. A late successful automatic retry MUST use the original occurrence date. Before attempting a confirmation purchase, Zunera MUST retain the owner's validated one-occurrence amount, date, card, and category choices on the occurrence independently of the financial mutation. Only one confirmation attempt per occurrence MAY own those choices at a time; a competing distinct action MUST wait or receive a retryable conflict and MUST NOT overwrite the active attempt. If recording fails, the occurrence MUST have no partial financial effect, remain actionable, and show those retained choices; a retry with omitted choices MUST reuse them, including the chosen purchase date. The owner MAY explicitly revise those choices on retry subject to current validation, without changing the rule or original occurrence snapshot. A successful purchase and the recorded occurrence MUST reflect the same winning amount, date, card, and category. An interrupted attempt MUST be recoverable after checking that no purchase was committed. Ordinary card statement and recognition rules apply to the date used for the successful purchase.
- **FR-028**: Recurrence list and detail responses MUST expose an owner-scoped count of card occurrences in `expected`, `awaiting_over_limit`, or `failed` states. The list MUST visibly mark a rule when this count is positive, using text as well as color, and refresh the mark after occurrence actions. Account rules and card rules with only recorded or dismissed occurrences MUST remain unmarked.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Every changed action MUST require authentication and owner-scoped authorization across recurrence, card, category, occurrence, purchase, statement, and derived financial values; unauthorized feedback MUST not reveal their existence or details.
- **SQR-002**: Submitted destination, amount, schedule, category, card, generation choice, occurrence action, and lifecycle change MUST be validated before financial effect. Existing over-limit confirmation and purchase restrictions remain enforceable.
- **SQR-003**: Automated coverage MUST prove account-rule compatibility, card-rule validation and ownership, both generation journeys, calendar and statement boundaries, retries/concurrency uniqueness, failure recovery, rule edits/lifecycle, over-limit behavior, refunds/corrections, and single recognition across Budget and Dashboard. Changed contracts and critical interface-to-financial-record journeys require automated coverage.
- **SQR-004**: Financial and display data remain owner-private. No full card number, CVV, PIN, new secret, upload, issuer connection, import, or external payment action is required or permitted.
- **SQR-005**: User-facing flows MUST work in Light, Dark, and System themes, responsive layouts, 200% zoom, keyboard use, and assistive technology, with financial states explained by text or equivalent non-color cues.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define protected behavior and contract for card-destination rules, generation choices, expected occurrence review, owner confirmation, source traceability, exactly-once purchase creation, validation, failure handling, and existing financial integrations. Verify these rules and unchanged account recurrence behavior before frontend delivery.
2. **Frontend** (`../zunera-frontend`): Extend existing recurrence create/list/detail/lifecycle journeys for destination and card choice, expected occurrence review, card identity and source links, actionable exception feedback, accessible responsive themes, and automated critical journeys using the established backend contract.

### Key Entities *(include if feature involves data)*

- **Recurring Transaction Rule**: Owner's repeating income or expense definition, schedule, immutable destination type, selected account or card, amount, category, generation choice where applicable, and lifecycle state; has no direct financial effect.
- **Scheduled Occurrence**: One rule and scheduled date, with a traceable expected, awaiting over-limit approval, dismissed, failed, or recorded outcome, any owner-confirmed actual purchase date, and original plus explicitly replaced card/category identities when applicable; distinct from the resulting financial record.
- **Credit Card Purchase**: One categorized obligation recorded from a card occurrence, with one installment and retained source link. Existing card purchase rules govern it.
- **Credit Card Statement and Installment**: Existing billing-cycle allocation and recognition of the generated purchase; determine obligation, expected spending, and realized spending.
- **Financial Account Transaction**: Existing account-destination occurrence record. It remains governed by ordinary pending/effective rules and never accompanies a card purchase for the same occurrence.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of representative users configure an eligible monthly card recurrence in under 3 minutes without assistance and can identify card, amount, generation behavior, and next date.
- **SC-002**: In 100 due-date cases spanning month-end, leap year, pause/resume, inclusive end date, and downtime, every eligible card occurrence has the correct date and no ineligible date creates a purchase.
- **SC-003**: In 100 retry, concurrent, and repeated-confirmation attempts per representative rule/date, no more than one purchase, one statement allocation, and one recognized expense result.
- **SC-004**: In 100 closing-date and billing-boundary examples, generated purchases have the same statement, due date, credit effect, and closing-month recognition as equivalent manually recorded single-payment purchases.
- **SC-005**: In 100 lifecycle and exception cases, rule edits, pauses, endings, card/category unavailability, failures, corrections, and refunds preserve every recorded purchase and correct historical totals.
- **SC-006**: At least 90% of representative users identify whether a charge is expected, recorded in Zunera, or externally unverified, and distinguish recurrence from installments, within 30 seconds.
- **SC-007**: For a list of 1,000 owned rules, 95% of recurrence-list and detail views become usable within 2 seconds under representative conditions; account-rule behavior remains unchanged.

## Assumptions

- Project numbers are Transactions 004, Recurring Transactions 006, Dashboard 008, Budgets 009, and Credit Cards 010. This extension supersedes only Credit Cards 010's initial exclusion of recurring-card purchases; its other financial rules remain authoritative.
- Existing recurrence schedules, start/end eligibility, missed active date catch-up, paused-date skipping, category restrictions, and lifecycle states remain defaults for both destinations.
- An unconfirmed card occurrence is a scheduled Zunera expectation, not a pending account transaction, recorded card purchase, issuer authorization, or externally verified charge. No external verification state is added.
- Once a purchase exists, its single installment is the Budget and Dashboard spending source. An unconfirmed occurrence is not eligible Budget Expected spending under existing rules, though it may appear as Dashboard upcoming activity.
- A card can be archived but historical purchases remain readable; a deleted or unavailable card cannot accept new purchases. No card payment credentials are collected.
- Same-type account or card changes affect only future unrepresented dates; changing destination type requires ending the old rule and creating another. Generated records retain historical source and meaning.
- Over-limit purchases retain explicit user confirmation; automatic processing never silently approves them and instead leaves an actionable occurrence awaiting the owner.
- Direct issuer charging, automatic imports, issuer verification, automatic statement payment, new installment management, and an independent subscriptions module are out of scope.
