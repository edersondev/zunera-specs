# Research: Recurring Transactions

## Decision: Source-link ordinary transactions

**Rationale**: A recurring occurrence is already an income/expense transaction.
Nullable source-rule and scheduled-date fields preserve origin while retaining
the existing pending/effective, removal, restore, ownership, and balance rules.

**Alternatives considered**: Separate occurrence ledger rejected because it
fragments history and correction; text-only source marker rejected because it
cannot enforce uniqueness or support rule navigation.

## Decision: Pending-only due creation and full eligible catch-up

**Rationale**: Clarification requires pending, never auto-effective, due
transactions. After downtime every eligible active schedule date is recovered
once as pending, preserving expected activity without unconfirmed balance change.

**Alternatives considered**: Effective on due date, skip missed dates, and
advance planned creation all rejected by clarification.

## Decision: America/Sao_Paulo date-only business calendar

**Rationale**: Zunera uses BRL and has no per-user time zone. Eligibility begins
at later of start date and creation business date, preventing past-start
backfill. Monthly missing ordinal uses month end; non-leap 29 February uses 28.

**Alternatives considered**: Server default zone risks deployment-dependent
behavior; per-user zone expands profile scope; skipping dates is surprising.

## Decision: Database-backed rule/date uniqueness

**Rationale**: A unique source rule plus scheduled date invariant protects
retries and concurrent due workers. Processor locks rule rows, creates pending
transaction/source link atomically, and treats duplicate conflict as processed.

**Alternatives considered**: In-memory de-duplication and content matching fail
for concurrent work or legitimate identical transactions.

## Decision: Archive pauses; end date ends

**Rationale**: Clarifications require account/category archive to automatically
pause rule with repair reason, and inclusive end date passing to automatically
end it. History is never changed; owner explicitly resumes after repair.

**Alternatives considered**: Active-but-blocked is ambiguous; archive ending is
too destructive; manual-only end creates active-but-finished state.

## Decision: Reuse idempotency convention and separate recurrence resource

**Rationale**: User mutations persist owner-scoped operation/fingerprint/result;
exact retry replays, changed key returns conflict. Due processor relies on
rule/date uniqueness. Recurrence gets its own resource and a paginated
occurrence collection; transactions and mixed history add nullable
`recurrence_source`, not a new movement type.

**Alternatives considered**: Client-only submit guard is insufficient; separate
history endpoint or transaction type fragments existing financial meaning.

## Decision: Reuse frontend transaction/transfer boundaries

**Rationale**: Axios owns transport, Pinia owns feature state, route view
composes focused components, and existing currency/date/account/category patterns
are reused. Element Plus controls plus semantic tokens meet design guidance.

**Alternatives considered**: API calls in components and one giant view violate
constitution and component-responsibility guidance.
