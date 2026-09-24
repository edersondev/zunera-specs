# Data Model: Recurring Credit Card Purchases

This extends the existing recurrence and credit-card models. Monetary values remain integer BRL centavos. Calendar dates use the existing Sao Paulo recurrence and card date rules. All reads and mutations are scoped to the authenticated owner.

## Recurring transaction rule (extended)

| Field | Meaning and validation |
|---|---|
| `id`, `user_id` | Existing immutable rule identity and owner. |
| `destination_type` | `financial_account` or `credit_card`; assigned at creation and immutable thereafter. Existing rows are `financial_account`. |
| `financial_account_id` | Required owned active account for account rules; absent for card rules. Historical archived association remains readable. |
| `credit_card_id` | Required owned active card for card rules; absent for account rules. Historical archived association remains readable. |
| `generation_mode` | `automatic` or `confirmation` for card rules, default `automatic` on new card creation. Account rules retain their established pending-transaction behavior and need no new input. |
| Existing `type`, `category_id`, `amount_centavos`, `description`, `notes`, `frequency`, `start_date`, `end_date`, `state`, `paused_reason`, `schedule_cursor` | Retain current formats and bounds. Card destination requires expense and active owned expense category. A missing or unavailable card/category uses the existing paused lifecycle with an actionable association reason. |

**Invariant:** Exactly one destination ID matches `destination_type`. A destination-type switch is rejected even if the new ID is otherwise valid. Same-type destination edits, schedule edits, generation-mode edits, and amount/category edits affect only future unrepresented dates. Existing `transactions` source records remain unchanged. Migration must make `financial_account_id` conditionally nullable, backfill current rows, preserve keys/indexes, and validate the exclusive destination rule in application code and a database constraint where supported by the existing database setup.

## Card scheduled occurrence (new)

One row represents an eligible card rule and scheduled date, whether or not a purchase was ultimately recorded.

| Field | Meaning and validation |
|---|---|
| `id`, `user_id`, `recurring_transaction_id` | Owner-scoped occurrence identity and source rule. |
| `scheduled_date` | Existing calendar calculator output; never overwritten by an actual purchase date. Unique with `recurring_transaction_id`. |
| `generation_mode_snapshot`, `scheduled_amount_centavos`, `description_snapshot`, `notes_snapshot`, `category_id_original`, `credit_card_id_original` | Values presented when the occurrence first became due. Immutable when the rule changes. |
| `card_identity_snapshot`, `category_name_snapshot` | Non-sensitive name/institution/last-four and category display values preserve the original identity even if an association later becomes unavailable. No full card number or credentials are stored. |
| `state` | `expected`, `awaiting_over_limit`, `failed`, `dismissed`, or `recorded`; workflow only, not a purchase or statement state. |
| `actual_amount_centavos`, `actual_purchase_date` | Latest validated owner choices for confirmation mode, retained before purchase creation and visible after a failed attempt; nullable until first confirmation, when omitted values default to scheduled amount/date. An omitted value on retry reuses its retained choice. An explicit revision is revalidated. Actual date must be no later than the current São Paulo business date. Automatic mode always uses scheduled amount/date. |
| `credit_card_id_override`, `category_id_override` | Optional explicit one-occurrence owner choices, retained before a confirmation purchase attempt and reused on retry unless explicitly revised. Validate ownership and eligibility on each attempt. Original snapshot fields remain unchanged. |
| `failure_code`, `last_attempt_at`, `recorded_at`, `dismissed_at`, `created_at`, `updated_at` | Retry and audit metadata. Failure detail is privacy-safe and contains no card credential. |

**Relationships and indexes:** Many occurrences belong to one rule; at most one purchase belongs to one occurrence. Unique `(recurring_transaction_id, scheduled_date)` reserves the date permanently. Index `(user_id, state, scheduled_date)` supports owner review/upcoming queries. A rule edit, pause, or end never deletes an occurrence. Existing due occurrences remain actionable after pause/end. A dismissed or recorded occurrence never returns to expected merely because processing is retried.

**State transitions:**

| From | Event | To | Financial effect |
|---|---|---|---|
| New automatic | Successful generation | `recorded` | One purchase and one installment, committed atomically. |
| New automatic | Credit limit requires owner approval | `awaiting_over_limit` | None. |
| New automatic | Retryable recording error | `failed` | None. |
| New confirmation | Due date presented | `expected` | None. |
| `expected` | Owner confirms, including optional actual values/replacements | `recorded` | One purchase and one installment. |
| `expected` | Validated confirmation purchase fails | `failed` | None; submitted one-occurrence choices remain visible and retryable. |
| `expected` | Owner dismisses | `dismissed` | None; date stays reserved. |
| `awaiting_over_limit` | Owner approves after current-credit recheck | `recorded` | One purchase and one installment. |
| `awaiting_over_limit` | Owner dismisses | `dismissed` | None. |
| `failed` | Safe automatic retry succeeds, blocks on credit, or fails again | `recorded`, `awaiting_over_limit`, or `failed` | Financial effect only on `recorded`. |
| `failed` confirmation | Owner retries with retained or explicitly revised choices, or dismisses | `recorded`, `failed`, or `dismissed` | Financial effect only on `recorded`; an over-limit conflict keeps the item actionable without silently approving it. |

If a transaction rolls back before reserving a failed occurrence, the unchanged schedule cursor makes the date eligible on the next run. A failed state is used only when the failure can be stored safely outside the failed purchase transaction. For confirmation, persist validated owner choices in a separate workflow step before the purchase transaction; a rollback of financial work cannot erase the chosen date or create a partial purchase. Retry rechecks current card/category and credit eligibility. A concurrency conflict is resolved by rereading the unique occurrence/purchase, never by creating a second record.

## Credit card purchase (extended)

| Field | Meaning and validation |
|---|---|
| Existing purchase fields | Existing card, owner, category, description, notes, purchase date, total amount, cancellation/refund state, and credit effects. |
| `recurring_card_occurrence_id` | Nullable for manual purchases; unique and required when created from a card occurrence. Its occurrence supplies source rule and scheduled date. Immutable after purchase creation. |

Every recurring purchase uses `installment_count = 1`, the existing statement allocator, limit rules, reconciler, correction/refund/cancellation rules, and purchase resource. The purchase source link remains even if the purchase is corrected, cancelled, or refunded; this never reopens its scheduled occurrence. No ordinary `transactions` row is created for a card occurrence. A foreign-key relationship and same-owner validation prevent cross-owner links; purchase creation and occurrence `recorded` transition commit together.

## Existing related entities

| Entity | Relationship and unchanged meaning |
|---|---|
| Financial account transaction | The financial record for account rules only; existing unique `(recurring_transaction_id, scheduled_date)` and pending/effective rules remain. |
| Credit card installment | Exactly one for each recurring card purchase. It is the sole Expected/Realized budget and expense source, based on its statement state and closing month. |
| Credit card statement | Assigned by the existing purchase date and billing-cycle calculator. Closing day is inclusive; backdated purchases restate an existing statement without replaying effective payments. |
| Credit card and category | Owned active associations are required for new rules and purchase creation. Archive/unavailability pauses future rule generation. Explicit eligible one-occurrence overrides do not change the template. Historical labels come from occurrence snapshots if an association cannot be loaded. |

## Derived projections and integrity

- A rule and an unconfirmed occurrence are scheduling facts. Neither changes an account balance, credit limit, card obligation, Budget Expected/Realized spending, or realized Dashboard totals.
- Upcoming Dashboard activity projects one eligible future card date or due unconfirmed occurrence. A recorded occurrence suppresses its corresponding projected item; its purchase appears once in recent/card activity.
- Once the purchase exists, the single installment supplies spending in the statement-closing month: Expected while Open, Realized after closing. Existing card credit/refund/correction rules adjust that result. Statement payment settles an obligation and does not create category spending.
- Account recurrence reporting stays on its existing ordinary transaction path. Transfers remain excluded from income and expenses.
- Queries must select card occurrence/purchase data in owner-scoped batches, with no per-rule occurrence or card lookups on list pages; the 1,000-rule/2-second target is checked under representative data.

## Migration and compatibility order

1. Add destination fields to rules, preserve and backfill all existing account rows, then conditionally relax account ID nullability.
2. Add the card occurrence identity/state/snapshot store and unique source reference on purchases; no historical purchases are converted.
3. Deploy reads that accept account and card rules and preserve old account-only request behavior before enabling card creation or due processing.
4. Add source-aware purchase orchestration, actions, and projections. Rollout tests verify current account rules, current card purchases/statements, and idempotency records remain valid.
