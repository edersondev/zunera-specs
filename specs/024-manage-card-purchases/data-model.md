# Data Model: Manage Credit Card Purchases

## Ownership and cycle identity

Every persisted record is owner-scoped through the card and `user_id`. Financial amounts are integer BRL centavos. A selectable statement cycle is identified by `(credit_card_id, closing_date)`; a cycle can be offered before it has a statement row. Existing rows retain their stored period and due-date snapshots after billing-day edits. For a month without a row, the cycle is calculated with the card's current closing and due days. The service rejects a target whose closing date is not the canonical date for its card/month or is before the purchase date.

The business day is calculated in `America/Sao_Paulo`. Closing date is inclusive: a cycle closes on the following business date. Statement options and mutations use the same resolver so a preview and save cannot disagree. Windowed options or a requested month permit any eligible future cycle without materializing empty statements during a read.

## Existing entities and changed attributes

### CreditCardPurchase

| Attribute | Rule |
| --- | --- |
| Existing owner, card, category, amount, date, description, notes, count | Unchanged by a statement move; existing purchase validation remains. |
| First statement choice | Manual create input `statement_closing_date`; required only when date-derived cycle is closed, otherwise absent. It is not a replacement for `purchase_date`. |
| `recurring_card_occurrence_id` | Remains source link for recorded recurrence purchases; deletion marks the occurrence before removing this row. |
| Deletion | Physical removal after confirming owner/card and zero linked credit events. Installments and move records are removed; related statement payments remain. |

Manual create includes the optional choice in its idempotency fingerprint. Automatic recurring purchase generation keeps its existing date-based cycle assignment.

### CreditCardInstallment

| Attribute | Rule |
| --- | --- |
| `sequence`, `amount_centavos`, purchase/card identity | Stable through moves; all amounts still sum exactly to the purchase total. |
| `credit_card_statement_id` | Changed for the selected installment or rebuilt for the full schedule. |
| `recognition_date` | Recomputed from the destination statement's closing date. |
| `credit_adjustment_centavos` | Recomputed from active linked credit-event applications after a move. |

For one-installment mode, closing dates ordered by `sequence` must be nondecreasing. Equal dates are allowed, so two installments may share a statement. Full mode assigns sequence 1 to the selected cycle and every later sequence to the next monthly cycle, replacing custom gaps/shared cycles. A full move with the same first cycle is valid when another installment changes.

### CreditCardStatement

| Attribute | Rule |
| --- | --- |
| Unique `(credit_card_id, closing_date)` | Prevents duplicate materialization of an empty selected cycle. |
| Stored period/closing/due dates | Immutable snapshot once materialized; reused when the card's billing days have since changed. |
| `original_amount_centavos` | Sum of current installment gross amounts. |
| `credit_adjustment_centavos` | Sum of current installment credit adjustments. |
| `paid_centavos` | Sum of effective, nonremoved payments; can exceed revised net charge after a move/delete. |
| `card_credit_applied_centavos` | Sum of active statement-level credit-event and payment-surplus applications. |
| `outstanding_centavos` | Derived as `max(0, net charge − paid − card credit applied)`; never negative. |
| `payment_surplus_centavos` | Derived as `max(0, paid − net charge)` after source installment-credit adjustments; exposed to explain excess settlement. |
| Status | Recomputed using existing Open/Closed/Partially paid/Paid/Overdue rules. |

An empty historical statement with move history or payments remains readable as a zero-charge statement. A newly selected empty cycle is materialized only as part of a successful create/move transaction.

### CreditCardStatementPayment

Effective payments remain linked to their original statement and financial account. Move/delete never edits their amount, status, account, or account-balance effect. Later payment update/remove/restore operations trigger card-credit/surplus reconciliation. Pending or removed payments contribute no surplus.

### CreditCardCreditEvent and CreditCardCreditApplication

Deletion is blocked when any credit event refers to the purchase. Moves keep the event's amount, reason, date, and source purchase, but its active installment/statement applications must follow revised allocations. Add nullable `superseded_at` to application rows: superseded rows remain audit history; active rows alone contribute to installment adjustment, statement credit, and card credit. Reconciliation supersedes and replaces affected active rows in one transaction.

## New entities

### CreditCardPurchaseMove

One record per installment whose statement changes during a confirmed move.

| Field | Rule |
| --- | --- |
| `id`, `user_id`, `credit_card_id`, `credit_card_purchase_id`, `credit_card_installment_id` | Required owner/card/source identity; purchase and installment foreign keys cascade on permanent deletion. |
| `mode` | `full_schedule` or `one_installment`. |
| `source_statement_id`, `destination_statement_id` | Existing same-card statement rows; retain both rows while move history exists. |
| Source/destination closing date, period, due date snapshots | Immutable display of what moved even if statement presentation later changes. |
| `moved_at`, `idempotency_key` or mutation reference | Time and one confirmed operation identity; unique operation plus installment prevents duplicate history on retry. |

Read through the purchase while it exists. Full-schedule move records only installments whose statement actually changed. A later hard delete removes these records with the purchase.

### CreditCardPaymentSurplusApplication

One active allocation of retained payment surplus from a source statement to an unpaid destination statement. This is separate from purchase-linked credit events.

| Field | Rule |
| --- | --- |
| `id`, `user_id`, `credit_card_id` | Required owner/card identity. |
| `source_statement_id`, `destination_statement_id` | Same-card statements; source differs from destination. |
| `amount_centavos` | Positive, never exceeds source's currently available surplus or destination's outstanding charge. |
| `applied_at`, `superseded_at` | Current versus historical allocation; only active rows count. |

For each source statement, active applications plus residual credit equal `payment_surplus_centavos`. Residual contributes to card credit. A card-wide reconciliation chooses destinations by existing oldest-unpaid order, due date then statement ID. If a later payment change or charge changes the surplus, prior active rows are superseded and replaced deterministically. No account transaction is created.

### RecurringCardOccurrence extension

Add nullable `purchase_deleted_at`. Keep `state=recorded` and scheduled-date uniqueness, set marker when its purchase is deleted, and expose `purchase_id=null` with a deleted indicator. The source rule and sibling occurrences remain intact; retries/catch-up treat the occurrence as already represented and never regenerate its purchase.

### CreditCardMutationRequest handling

Keep owner, idempotency key, operation, fingerprint, and deleted purchase's opaque target ID. Delete response contains only deleted ID and affected statement IDs; identical replay returns it without a second effect. On deletion, overwrite cached purchase-specific create/update/move responses with a deleted-resource outcome and remove purchase descriptive/financial payloads. Old keys remain claimed, so an old create cannot recreate the purchase. Recurrence mutation snapshots that embed the deleted purchase must receive the same scrub treatment.

## Reconciliation invariants and state transitions

```text
Current installment allocation
  -> gross statement charges
  -> active source installment credits
  -> net statement charges
  -> retained effective payments
  -> payment surplus on overpaid source statements
  -> active general card-credit applications to oldest unpaid statements
  -> per-statement nonnegative outstanding + residual card credit
```

`used_credit = sum(statement outstanding)` across all card statements, including Open obligations. `card_credit = unapplied credit-event value + unapplied payment surplus`. `available_credit = card limit − used_credit + card_credit`. Sum all calculations in centavos; never count a payment as a categorized expense. Card reconciliation must be idempotent and run under deterministic card/statement locks after create, move, delete, credit event, and payment lifecycle mutations. Emit statement/budget notification projections for all old and new recognition months and affected statement IDs.

| Transition | Persistent result |
| --- | --- |
| Manual backdated create | New purchase/installments assigned to explicitly selected first cycle; existing open-date create remains automatic. |
| Full-schedule move | Same purchase/installment IDs, consecutive destination cycles, move records for changed rows. |
| One-installment move | One installment changes statement; schedule remains in nondecreasing closing-date order. |
| Delete | Purchase, installments, and move history removed; payment rows, card/statement records, minimal idempotency tombstones, and any occurrence marker remain. |
| Retry/stale request | Same-key same-payload replay or specific conflict; no second financial effect. |

## Validation and ownership

- Every choice must be same-card, canonical for its month, and close on/after purchase date; a full schedule must have valid consecutive target cycles.
- Manual closed-cycle create requires an explicit choice. Open-cycle create rejects a supplied override and retains automatic allocation.
- One-installment move requires an owned installment of the purchase, a different target, and nondecreasing neighboring statement dates. Full move requires at least one changed installment.
- Move preview and mutation recheck ownership, linked records, current statement metadata, and source/target balances under lock. A changed preview fingerprint yields conflict without mutation.
- Delete requires owner access and no linked credit event of any application state. Invalid/foreign records reveal no private details. All mutations use the existing idempotency-key convention and atomic rollback on failure.
