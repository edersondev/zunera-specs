# Research: Recurring Credit Card Purchases

## Existing behavior and decision record

### 1. Extend the existing recurrence rule

**Decision:** Add an immutable `destination_type` to the existing recurring rule. Existing rows and requests without that field mean `financial_account`. Card rules require an active owned card and expense category, and expose `generation_mode` (`automatic` by default, or `confirmation`). An account rule continues using its existing pending-transaction path.

**Rationale:** The recurrence form, lifecycle, calendar calculator, daily job, and account transaction source already exist. Keeping one rule type preserves their behavior and permits account records to migrate without user action. The current `financial_account_id` is mandatory, so the migration must make it nullable only when a card destination is present and backfill/derive the default safely.

**Alternatives considered:** A second subscription module duplicates scheduling and lifecycle rules. A generic polymorphic destination obscures validation and makes the two allowed destinations harder to enforce.

### 2. Persist each card occurrence separately from a purchase

**Decision:** Introduce an owner-scoped card occurrence with a unique `(recurring_transaction_id, scheduled_date)` identity and workflow state `expected`, `awaiting_over_limit`, `failed`, `dismissed`, or `recorded`. Snapshot the rule's payment card and non-sensitive display identity, category and name, description, note, amount, generation mode, and scheduled date when the occurrence is presented. A purchase has a nullable, unique source occurrence reference. Account occurrences retain their existing transaction source identity.

**Rationale:** A confirmation request needs a durable expected item before there is a purchase. Over-limit and failed attempts also need a retryable, visible identity. The source uniqueness survives retries, rule edits, refunds, and cancellation. A presented due occurrence can remain actionable after the rule is paused or ended.

**Alternatives considered:** Deriving every expected occurrence from the current rule would let edits rewrite history and make dismissal impossible to audit. Relying only on an API idempotency key cannot prevent duplicates from scheduler retries or multiple keys.

### 3. Make due processing atomic and recoverable

**Decision:** Keep the existing Sao Paulo daily due scheduler and `RecurringScheduleCalculator`, but replace the card path's current one-transaction-for-the-rule backlog with a transaction for each scheduled date. Under the owner-scoped rule lock for that date, reserve its occurrence, branch on generation mode, and advance the cursor only when its outcome is durably represented. Confirmation mode stops at `expected`. Automatic mode invokes the existing card-purchase rules for each reserved date. A recoverable purchase error is persisted as `failed` outside its rolled-back purchase transaction, then later eligible dates continue; if the failure identity itself cannot be persisted, stop this rule with its unrepresented date still eligible for retry. Track `failed` and `awaiting_over_limit` items independently of later dates. Repeated processing must inspect existing occurrence state before any mutation. DB uniqueness is the final concurrency guard. Account processing retains its current behavior.

**Rationale:** The current processor locks a rule but wraps the whole backlog in one transaction. That would roll back an earlier successful card date if a later date fails. Per-date commits preserve earlier obligations and let a safely represented failure coexist with later outcomes; the calendar calculator still handles short months/leap dates.

**Alternatives considered:** One transaction for an entire backlog would roll back successful dates after a later failure. Advancing the cursor before persisting an occurrence would silently skip a charge. A queue or new scheduler adds no necessary capability.

### 4. Reuse the card purchase mutation and over-limit rules

**Decision:** Add an internal, source-aware entry to `CreditCardPurchaseService` (or shared domain operation) so occurrence recording uses the same ownership checks, active card/category checks, credit calculation, billing-cycle assignment, single-installment creation, idempotency, and obligation reconciliation as manual purchases. Orchestration must use one atomic transaction with a consistent lock order for rule/occurrence/card; the purchase source uniqueness and state transition commit together. The browser confirms an occurrence through a recurrence action, never through manual purchase creation. Automatic over-limit leaves the occurrence `awaiting_over_limit` without a purchase; owner approval rechecks current available credit and uses the existing explicit `confirm_over_limit` semantics with a fresh idempotency key.

**Rationale:** `CreditCardPurchaseService` already owns purchase invariants and typed over-limit conflicts. Laravel 13 documentation supports row locks inside transactions and transaction retry attempts for deadlocks; the database constraint still provides the at-most-one guarantee. One route through the purchase rules prevents differences in card totals.

**Alternatives considered:** Calling the public card endpoint from the scheduler would lose atomic occurrence linkage. A second purchase implementation risks mismatched statement and credit results. Silent automatic over-limit approval violates the clarified business rule.

**References:** `../zunera-backend/app/Services/RecurringTransactions/RecurringOccurrenceService.php`, `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php`, [Laravel database transactions](https://laravel.com/docs/13.x/database#database-transactions), [Laravel pessimistic locking](https://laravel.com/docs/13.x/queries#pessimistic-locking).

### 5. Apply the existing billing and recognition model

**Decision:** Automatic purchase date is the scheduled date; confirmation defaults to that date but accepts one valid actual date no later than the current São Paulo business date. This confirmation restriction is narrower than manual card purchase date validation because it records a charge the owner says occurred. The existing billing-cycle calculator assigns the statement, including a closing-date purchase on that closing date. Backdated purchases use the original statement even if closed or paid; existing reconciliation restates its totals and credit while preserving effective payments. A card rule or unconfirmed occurrence does not create an account transaction or budget expense. Once purchased, the single card installment is the only spending source: expected while statement is open, realized in its closing month thereafter. Dashboard upcoming activity may show the scheduled expectation once; recent activity shows the recorded purchase once and suppresses the same expected item.

**Rationale:** Credit card budget projection and financial history already recognize installment amounts by statement closing period. Credit card obligation reconciliation already recomputes statement state after card mutations. The generic Dashboard transaction feeds need an explicit card-recognition integration and deduplication check; otherwise the spec's Dashboard behavior would be only partly met.

**Alternatives considered:** Recognizing a recurrence rule as spending would double count its later purchase. Posting an account expense or charging a statement payment as new spending would change established accounting.

### 6. Preserve historical meaning through edits and exceptions

**Decision:** Before an owner-initiated card rule edit, pause, or end changes the template or eligibility, catch up all eligible active dates already due through the current São Paulo business date using the old rule definition and the per-date transaction path. Then lock and verify that no eligible due date remains unrepresented before applying the mutation. A concurrent edit or processor change requires rechecking; if a due date could not be durably represented, return a retryable conflict and leave the rule unchanged. Earlier date commits remain valid. A represented failed or awaiting-over-limit date retains its original snapshot while the rule may change. Existing snapshots and purchases remain fixed. An unavailable current card or category auto-pauses the active rule under its existing availability semantics; a previously presented expected occurrence can use an explicit eligible card/category replacement for that occurrence, retaining original IDs. A due expected item stays confirmable or dismissible after pause/end. Purchase corrections, cancellations, and refunds use existing card operations and leave the source date reserved. Failed generation exposes a retryable reason without partial financial effect.

**Rationale:** A recurrence is a template and the purchase is a financial record. Separate identities let the user repair a single occurrence without rewriting the template or statement history.

**Alternatives considered:** Mutating all open occurrences with rule edits would erase what was originally expected. Regenerating after a refund or cancellation would create a duplicate obligation.

**Confirmation retry decision:** Claim the occurrence for one confirmation attempt before persisting its validated one-occurrence amount, date, card, and category choices. A competing distinct attempt cannot revise those choices while the claim is active and gets a retryable conflict. The purchase must read the claimed choice version under the occurrence lock and commit with the `recorded` transition; its fields and the occurrence's actual fields must match. If the financial mutation fails, keep the occurrence in `failed` with the claimed choices and release the claim. A bounded, owner-scoped stale-claim recovery rechecks for a committed purchase before allowing another attempt after an interrupted worker. Omitted retry fields reuse the retained choices; explicit changes are validated again. An automatic failed occurrence instead keeps its scheduled values. This preserves the owner's chosen actual date without changing the rule template or allowing a losing request to overwrite a recorded purchase's audit values.

### 7. Extend the current user experience and contract

**Decision:** Add destination/card/generation controls to the existing recurrence form; show card identity and workflow state in existing list/detail and occurrence views. Replace unsupported-card text with contextual guidance explaining recurring purchases versus installments. Extend the existing recurrence API and resource, add owner-scoped confirm/dismiss actions, and include recurrence source in purchase/card statement activity. Continue using Axios service, Pinia store, `<script setup>`, Element Plus controls, and the four project design documents. Build backend contract, authorization, validation, and tests before frontend work.

**Rationale:** The frontend already has recurrence actions and an over-limit confirmation pattern, and the user explicitly requires one management experience. The credit-card card-identity formatter and existing Light/Dark/System tokens can be reused.

**Alternatives considered:** A separate page would fragment rule management. Calling the card manual-purchase action would not identify the occurrence and would make duplicate prevention depend on UI behavior.

## Resolved unknowns

The business decisions in `spec.md` resolve generation default, variable amount/date, future-date rejection on confirmation, over-limit approval or dismissal, per-occurrence replacement, due occurrence lifecycle, late statement allocation, and immutable destination type. No product clarification remains for task generation.
