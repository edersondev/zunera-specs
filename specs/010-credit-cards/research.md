# Research: Credit Cards

## Distinct card domain and settlement

**Decision**: Model credit cards separately from ordinary financial accounts.
Purchases create card obligations; statement payments are dedicated settlement
movements that debit a selected financial account without becoming ordinary
categorized expense transactions.

**Rationale**: Reusing `FinancialAccount` or creating a normal expense
`Transaction` for a card payment would misrepresent available cash or count the
same purchase twice in spending totals. Existing account balances can receive a
focused effective settlement delta while the card retains payment history.

**Alternatives considered**: Credit-card account type (collapses debt and cash);
ordinary payment expense transaction (duplicates category spending); no account
movement record (cannot restate balance correctly).

## Cycle, due date, and recognition

**Decision**: Use America/Sao_Paulo business dates. Closing day is inclusive;
missing month days clamp to month end; due date is the first configured due-day
occurrence after actual closing. Each installment recognizes spending in calendar
month containing its statement closing date. It is pending while statement is
open and effective when statement closes; only effective, non-removed net
installments enter realized Budget/Dashboard/history totals.

**Rationale**: This makes purchase, obligation, recognized spending, and cash
settlement distinct and predictable around February and month/year boundaries.
It matches monthly-budget source lifecycle without using payment date.

**Alternatives considered**: Purchase-date recognition (wrong for installments);
payment-date recognition (duplicates/defers spending); due-date recognition
(changes spending month without billing reason).

## Billing configuration changes

**Decision**: Preserve every allocated statement's period, closing date, due
date, and installment assignment. A card closing-day or due-day update applies
only to purchases recorded after that update.

**Rationale**: A user can correct future card settings without silently changing
already communicated statements, current obligations, or recognized spending.

**Alternatives considered**: Recalculate open/future statements (rewrites user
expectation and budget attribution); prohibit later configuration changes.

## Credit limit, refunds, and card credit

**Decision**: Full purchase total reserves available credit immediately.
Effective payments and credit-event adjustments reduce used principal. Credit
event first offsets source unpaid installments; excess becomes separately visible
card credit. Card credit automatically settles oldest unpaid statement by due/
closing order, with remaining credit increasing usable capacity above stated
limit.

**Rationale**: Produces exact card liability, preserves paid-statement refund
meaning, and avoids negative statement balances or hidden issuer credit.

**Alternatives considered**: Reserve only current installment (understates
commitment); cap availability at limit (hides usable refund credit); require user
to choose credit allocation (creates avoidable stale debt).

## Traceable corrections

**Decision**: Direct purchase edit is allowed only while all installments are in
Open statements. Cancellation, full/partial refund, and post-closing correction
create immutable credit events with ordered allocation applications; no original
purchase is deleted.

**Rationale**: Closed history remains auditably stable while projections can
restate from source events.

**Alternatives considered**: Silent edit/delete (destroys history); refunds only
(cannot correct a closed error); manually entered negative expense (breaks card
and cash-domain separation).

## Atomicity, idempotency, and account safety

**Decision**: Every money mutation uses a persisted owner/key/fingerprint replay
record and one database transaction. Lock accounts in ascending id, then cards,
statements, purchases, and payments. Recheck limit/outstanding after locks.
Statement settlement follows ordinary expense-like account delta rules, without
inventing Transfer's no-negative-source restriction.

**Rationale**: Existing transaction/transfer idempotency and locked balance
reconciliation establish project precedent. Deterministic locks prevent double
payments, stale limit decisions, and lost correction effects.

**Alternatives considered**: Browser duplicate prevention only; optimistic
updates without server locks; applying Transfer's source restriction despite
ordinary expense behavior being distinct.

## Derived integrations and performance

**Decision**: Extend server-side Budget, Dashboard, and Financial History source
projections with recognized card installments. Card payments remain settlement
only. Add an additive dashboard card projection for obligations/due statements;
do not alter active-account balance allocation. Keep zero current statement as a
derived view. Measure 10,000 combined movements before adding indexes or cache.

**Rationale**: Existing projections own financial truth, and a separate additive
card section avoids breaking account-balance semantics or client-side arithmetic.

**Alternatives considered**: Persist aggregates; aggregate browser history;
include payment as expense; introduce cache/index before measured need.
