# Research: Account Transfers

## Decision: Use dedicated transfer record with two account associations

**Rationale**: Transfer is one event with equal and opposite account effects.
Keeping source and destination together prevents independent edits from
splitting event and keeps it distinct from income and expense transactions.

**Alternatives considered**:

- Two independently editable income/expense records: rejected because either
  side could be changed or removed alone.
- Classifying transfer as income or expense: rejected because it corrupts
  income, expense, result, and net-worth reporting.
- Adding transfer to existing transaction record without separate rules:
  rejected because transfer needs two accounts and source-balance validation.

## Decision: Store amounts and balances as integer centavos

**Rationale**: BRL values from R$ 0,01 through R$ 999.999.999,99 map exactly to
integer centavos. This matches Financial Accounts and Transactions and preserves
exact arithmetic.

**Alternatives considered**:

- Floating point: rejected because accumulated rounding drift breaks balance
  invariants.
- Decimal strings: rejected because it creates second money convention.

## Decision: Reconcile both account effects atomically by delta

**Rationale**: Account current balances are materialized balances of record. For
every mutation, calculate each affected account before/after transfer effect,
then apply current minus before effect plus after effect inside one database
transaction. Lock all affected account rows in ascending identifier order before
validating or updating them; deterministic order prevents opposing transfers
from deadlocking. Laravel 13 documentation recommends wrapping pessimistic locks
in database transaction; failures roll back and release locks.

**Alternatives considered**:

- Update source then destination separately: rejected because partial failures
  create or destroy money.
- Recalculate all history on every write: rejected because exact delta is known.
- Lock accounts in request order: rejected because opposing transfers can
  deadlock.

## Decision: Reject an effective transfer that overdraws source

**Rationale**: Clarification A requires every effective transfer to leave source
at or above R$ 0,00. Validate proposed reconciled balance, not raw balance, so
update reducing or reversing existing debit frees prior effect first. Pending
transfer may be retained until funds exist. Destination proposed balance must
also remain within Financial Accounts existing maximum.

**Alternatives considered**:

- Permit negative balances for all accounts: rejected by clarification.
- Introduce per-account overdraft eligibility: rejected because it expands
  account model without user requirement.

## Decision: Reuse pending/effective status and recoverable removal

**Rationale**: Transfers follow Transactions: only effective non-removed
records count; pending records contribute zero; removal retains history and
reverses both effects exactly once. Removed transfers are read-only until
restored.

**Alternatives considered**:

- Always-effective transfers: rejected because planned future movements must not
  affect current balances.
- Permanent deletion: rejected because it harms financial history and recovery.
- A removed status value: rejected because removal is independent of financial
  status.

## Decision: Future transfers default to pending without silent status edits

**Rationale**: Past/current creation defaults to effective; future creation and
restoration default to pending and reject an effective request while the date
remains future. After the date is today or past, an explicit effective update
revalidates source funds. Moving already-effective transfer date into future
preserves its effect and returns notice rather than silently changing financial
state.

**Alternatives considered**:

- Silently change effective to pending on date edit: rejected because it
  silently changes balances.
- Allow future effective creation by default: rejected because planned movement
  would distort balances.

## Decision: Preserve archived associations but forbid new archived choices

**Rationale**: Existing transfer history must remain understandable. Embedded
source/destination summaries include lifecycle status. User may retain archived
side while correcting other fields, but new/replacement side must be active and
owned. Account archive never mutates transfer state or balance effect.

**Alternatives considered**:

- Freeze all transfers linked to archived account: rejected because genuine
  corrections would be impossible.
- Allow archived accounts for new transfer sides: rejected because archival
  removes normal operational choices.

## Decision: Require owner-scoped idempotency for every mutation

**Rationale**: Browser submit guard cannot protect retries, refreshes, or
concurrent requests. Persist owner-scoped key, operation, normalized
fingerprint, and completed response in same transaction as reconciliation.
Exact replay returns stored response; changed reuse returns typed conflict.

**Alternatives considered**:

- Client-only prevention: rejected because it cannot protect direct callers or
  network retries.
- Unique transfer-content constraint: rejected because identical valid transfers
  are allowed.

## Decision: Provide dedicated transfer CRUD plus a canonical mixed-history read endpoint

**Rationale**: Transfers need source/destination filters and lifecycle actions,
so have own resource. Clarification B requires existing financial/transaction
history to show transfers beside income and expenses. `GET /financial-history`
is the sole canonical mixed read projection, owned by FinancialHistoryController
and consumed by the existing transaction-history screen. The transaction-only
`GET /transactions` contract from feature 004 remains owned by its existing
controller and is not registered again or changed by this feature. Both read
surfaces keep mutations separate. Income/expense totals consume income and
expense movements only.

**Alternatives considered**:

- Dedicated transfer list only: rejected by clarification B.
- Reinterpret transfer as transaction type: rejected because it has no category
  and needs two balanced account effects.

## Decision: Name the aggregate owners and assert their transfer invariants

**Rationale**: FinancialHistoryService owns income, expense, and financial-result
totals; FinancialAccountService owns owned-account balance and net-worth
summaries. Transfer rows are excluded from the first service's income/expense
classification. The second service applies both account deltas, so an effective
transfer changes the two account balances but preserves their combined net-worth
total. Tests must exercise each currently exposed aggregate read surface; a
future reporting surface must apply the same movement-kind rule before release.

**Alternatives considered**:

- Treat generic report regression coverage as sufficient: rejected because it
  would not prove which aggregate owns each invariant.
- Make every account balance unchanged: rejected because a transfer must debit
  its source and credit its destination.

## Decision: Use established ownership, search, pagination, and UI boundaries

**Rationale**: Owner-scoped reads and privacy-safe not-found prevent disclosure.
Filters combine with AND semantics; normalized description/notes search ignores
case and accents; batches up to 50 and total count support 5,000 records. Vue
script setup owns local UI state, Pinia setup store owns shared feature state,
and Axios remains in transfer service.

**Alternatives considered**:

- Database collation only: rejected because accent behavior differs by engine.
- Client-side history pagination: rejected because it downloads too much.
- API calls in components: rejected by constitution.
