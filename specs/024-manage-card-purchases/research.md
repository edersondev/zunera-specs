# Research: Manage Credit Card Purchases

## 1. Statement identity and discovery

**Decision**: Identify a selected cycle by its `closing_date` plus the owned card. Expose a windowed statement-options query with a month cursor and direct month lookup. Return the date-derived cycle, whether it is closed on the São Paulo business date, and each option's period, closing date, due date, persisted statement ID when present, and open/closed state. Include empty future cycles; a requested month can be reached without loading every intervening cycle.

**Rationale**: Empty cycles have no statement ID. Existing `CreditCardStatement` rows retain historical billing dates after a card's billing-day change; these rows take precedence over a newly calculated cycle. `BillingCycleCalculator` already owns inclusive closing, month-end clamping, and due dates. Revalidate a selected closing date against the locked card and existing statement row before mutation.

**Alternatives considered**: Statement IDs alone cannot identify empty future cycles. A single unbounded list cannot support any future cycle efficiently. Client-side date arithmetic would diverge around billing-day changes and time zones.

## 2. Manual creation and recurring purchases

**Decision**: Extend only the manual create request and DTO with `statement_closing_date`. When the purchase date's derived statement has closed, require an explicit eligible value; when it remains open, reject an unexpected override and retain automatic allocation. Put the selection in the create idempotency fingerprint. Later installments follow consecutive cycles from the chosen first statement. Keep recurring occurrence purchase generation on its existing automatic date-based path.

**Rationale**: The selector belongs to the manual dialog. The current service has separate manual and recurrence entry points. The server must own the closed-cycle rule and selection validation.

**Alternatives considered**: Reusing an optional statement ID would fail for empty cycles. Applying the manual choice requirement to automatic recurrence generation would change behavior outside this feature.

## 3. Move preview and mutation

**Decision**: Add a read-only preview operation and an idempotent move operation. Both accept `mode` (`full_schedule` or `one_installment`), target closing date, and an installment ID for the one-installment mode. Preview returns affected installment source/destination snapshots, resulting statement charges, and a state fingerprint. Mutation recomputes that preview after locking and rejects a stale fingerprint. Preserve installment IDs, amounts, sequence, and purchase date. Full mode rebuilds consecutive cycles; one mode changes only the selected installment and validates nondecreasing closing dates.

**Rationale**: Closed and paid statement effects must be understandable before confirmation. Recomputing under locks prevents a stale preview from authorizing a different financial result.

**Alternatives considered**: Reusing the open-only purchase correction endpoint would bypass its lifecycle guard and mix unrelated field edits with reassignment. Updating statement foreign keys without a preview would hide payment and spending changes.

## 4. Traceable moves and historical credit adjustments

**Decision**: Store a move record per changed installment with owner, card, purchase, mode, source/destination statement identities, and time. Reconcile linked credit-event applications when a credited installment moves: retain event amount and reason, supersede prior active allocations, recompute installment and statement allocations, and keep superseded rows available for audit while the purchase exists. A later permanent deletion cascades move history, because the purchase must disappear.

**Rationale**: Current credit-application rows refer to both installment and statement. Moving only the installment would leave credit adjustment totals on the wrong statement. Earlier credit-event applications are treated as immutable history, so supersession records the change without rewriting their historical rows.

**Alternatives considered**: Blocking all credited moves would violate the requested general move behavior. In-place rewriting of credit-application history would obscure what changed.

## 5. Payment surplus and card credit

**Decision**: Add a payment-surplus application ledger sourced from the statement retaining its effective payments and applied to destination statements. Reconcile the whole affected card deterministically under a card lock after move, deletion, manual/recurring creation, credit event, and payment lifecycle changes. Effective payments keep their original statement and paying-account movement. After charge restatement, payment above that statement's net charge becomes a surplus source; apply it to oldest unpaid statements by existing due-date order, and expose any remainder as card credit. Compute used credit from the sum of nonnegative statement outstanding balances.

**Rationale**: Current card credit comes only from purchase-linked credit events, whose application rows require a credit-event foreign key. A deleted purchase cannot be used to fabricate such an event. The current global `net installments − payments` formula can understate remaining debt when one paid statement becomes overpaid. A separate ledger prevents double use of surplus and preserves cash history.

**Alternatives considered**: Leaving the surplus on the source statement hides card credit. Reassigning or reversing payments would invent cash movements. Creating a refund event contradicts permanent deletion. A single balance field cannot explain which later statement consumed surplus.

## 6. Permanent deletion and idempotency

**Decision**: Delete only after locking the owned card and purchase and confirming no linked credit events. Remove purchase, installments, and move history; retain statement payments. The delete response is a minimal summary with deleted ID and affected statement IDs, safe to replay with the same idempotency key. Before deletion commits, replace purchase-specific cached card and recurrence mutation response bodies with a minimal deleted-resource response while retaining keys/fingerprints; an old create or update key must not resurrect or reveal the purchase. Resolve a delete replay by its owner/key before trying to load the now-absent purchase. A different key for the deleted ID has no money effect and receives a missing-resource response.

**Rationale**: Card and recurrence mutation request records can store purchase response snapshots without foreign keys. Deleting database rows alone would leave purchase data in replay payloads. Existing create/update services load the target after replay, so they need an explicit deleted-target outcome.

**Alternatives considered**: Soft deletion does not meet permanent removal. Dropping all idempotency rows would allow old create requests to create another purchase. Retaining original response snapshots would violate the deletion outcome.

## 7. Recurring occurrence retention

**Decision**: Keep a recorded occurrence's identity and scheduled date, add a purchase-deleted marker, and remove the purchase link through the hard delete. The recorded occurrence must remain non-generatable; its detail shows that its purchase was deleted, without a broken purchase link. Rule and sibling occurrences stay unchanged.

**Rationale**: The purchase points to its occurrence, and the occurrence currently stays `recorded` after a purchase row disappears. An explicit marker makes that state understandable and avoids regeneration during retries or catch-up.

**Alternatives considered**: Deleting the occurrence could cause replayed scheduling. Returning a `recorded` occurrence with an unexplained null purchase is misleading.

## 8. Frontend composition and verification

**Decision**: Keep route views as orchestration surfaces. Extend the existing purchase form with server-backed statement options and add focused move and delete dialogs. Purchase-list and statement-line actions emit intent; a statement-line action preselects its installment. Use existing Axios service and Pinia card store mutation/refresh paths. Keep JavaScript `<script setup>` per current project convention, Element Plus controls, localized pt-BR/en text, semantic theme tokens, and no new package.

**Rationale**: Existing view, service, store, and dialog patterns already separate transport from presentation. Server-derived options avoid client-side billing disagreement. Project design documents require labelled forms, explicit destructive confirmation, focus return, responsive layout, and durable refreshed content.

**Alternatives considered**: Placing financial calculations in Vue would duplicate backend rules. A new global store or generic modal framework adds scope without a shared-state need.

## 9. Validation evidence

**Decision**: Cover contract and owner isolation, month-end and closing-day choice rules, both move modes after custom placements, linked-credit moves, paid-statement surplus, hard delete with repeat and old-key replay, recurrence tombstone, and exact once budget/dashboard/history projections. Run backend feature/unit tests and Pint before frontend work; then Vitest, lint, build, and isolated Playwright journeys for create/move/delete.

**Rationale**: The main failure modes are financial duplication, stale choices, and private-data exposure. Existing project commands and test layout support these checks.

**Alternatives considered**: UI-only tests cannot establish monetary invariants. Broad test-suite reruns are useful as final regression gates but cannot replace focused financial cases.
