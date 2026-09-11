# Research: Transaction Management

## Decision: Store monetary amounts as positive integer centavos

**Rationale**: The financial-account domain already stores balances as integer
centavos with a documented supported range. Amounts from R$ 0.01 to
R$ 999,999,999.99 map exactly to `1` through `99,999,999,999`, so income and
expense arithmetic never introduces rounding drift and never needs floating
point comparison.

**Alternatives considered**:

- Floating-point amounts: rejected because repeated additions and subtractions
  drift and break balance equality checks.
- Decimal strings: rejected because arithmetic then requires a decimal library
  and a second numeric convention alongside the existing centavos domain.
- Signed amounts with a separate direction field: rejected because the spec
  requires always-positive amounts whose direction comes from the type.

## Decision: Keep the materialized account balance and reconcile by delta

**Rationale**: `financial_accounts.current_balance_centavos` is already the
balance of record exposed by the resource and by the account summary endpoint.
Transactions reconcile it inside the same database transaction that changes the
transaction record, by computing the previous and the resulting effect and
applying the difference exactly once while the affected account row is locked.

**Alternatives considered**:

- Derive balances on read from initial balance plus transactions: rejected
  because the existing account contract returns a stored balance, and every list
  and summary read would become an aggregate query.
- Recompute the entire account balance on every write from all its
  transactions: rejected as an unnecessary full-table aggregate when the delta
  is known, and harder to reason about under concurrency.
- Unlockable "adjustment rows": rejected as an unrequested accounting model.

## Decision: Two financial states, pending and effective

**Rationale**: The clarified product rule is exactly two states. Only
`effective` transactions change balances, so every balance decision reduces to
whether a transaction is effective and not removed. Both directions of change
are allowed, and each transition applies or reverses the effect once.

**Alternatives considered**:

- A third planned/confirmed state: rejected by clarification as unnecessary
  pipeline depth.
- A single always-effective state: rejected because planned future movements
  must not distort current balances.
- A derived state from the date: rejected because the user must be able to mark
  a future-dated movement as already effective.

## Decision: Future-dated transactions always start as pending

**Rationale**: The spec forbids a future-dated transaction from changing a
balance until the user says the movement happened. Rejecting an explicit
`effective` request on a future date, instead of silently overriding it, keeps
the API honest and gives the user an actionable message.

Editing is a deliberate user action on a record whose state is already visible,
so an edit that moves an effective transaction's date into the future preserves
the effective state and its balance effect and explains the consequence instead
of silently pulling money out of a balance. The response carries a notice that
the transaction remains effective and still affects the balance, along with the
option to set it pending.

**Alternatives considered**:

- Silently forcing `effective` to `pending`: rejected because a caller would
  receive a different state than requested with no explanation.
- Allowing effective future transactions by default: rejected because a planned
  expense would incorrectly reduce the current balance.
- Rejecting future dates entirely: rejected because planned movements are a
  legitimate use case.
- Auto-reverting an effective transaction to pending when its date moves into
  the future: rejected because it silently changes a balance the user did not
  ask to change.
- Rejecting the date edit until the caller also sends a status: rejected as a
  confusing, unnecessary two-step contract.

## Decision: Model removal as a nullable `removed_at` timestamp

**Rationale**: Historical integrity and user recovery both require the record
to survive. A removed transaction keeps every field, stops contributing to
balances, leaves the normal history, and moves to a removed view where the owner
can restore it. Reconciliation logic treats "effectively counts" as
`status = effective and removed_at is null`.

**Alternatives considered**:

- Hard deletion: rejected because history, prior reports, and category/account
  associations would lose the record.
- A third `removed` value on the status enum: rejected because removing is a
  lifecycle decision orthogonal to whether the movement was financially
  effective, and a removed effective transaction must remember it was effective.
- A separate removed-transactions table: rejected as duplicate schema and
  migration burden with no behavioral gain.
- Editing a removed transaction in place: rejected because the record is outside
  the active history; a removed transaction is read-only until it is restored,
  which keeps one rule for users and prevents balance changes on a record the
  user believes is gone.

## Decision: Type and status are enums with explicit values

**Rationale**: `income` and `expense` are the supported types, and the enum
keeps the movement model extensible: a future transfer type becomes a new value
with its own validation and balance rules instead of a reinterpretation of
existing rows. The same applies to `pending` and `effective`.

**Alternatives considered**:

- Free-text type: rejected because it makes validation and reporting
  unreliable.
- Boolean `is_income`: rejected because adding a third movement type would
  invalidate stored data.
- A separate movement-types table: rejected as unneeded complexity for a closed
  enum known at design time.

## Decision: Ownership checks produce privacy-safe 404 responses

**Rationale**: The financial-account and category features already hide other
users' resources behind a not-found response. Transactions follow the same
rule, so a user cannot discover whether another user's account, category, or
transaction exists. Own archived resources are different: the user already
knows they exist, so those are rejected as validation errors with an
explanation.

**Alternatives considered**:

- 403 for another user's resources: rejected because it confirms existence.
- Validation error `exists` rules on the raw request: rejected because they
  confirm that a foreign identifier exists.
- Allowing a foreign account through service validation: rejected as a direct
  ownership violation.

## Decision: Allow keeping an archived association but never selecting one anew

**Rationale**: The clarified rule keeps old records editable. A user may fix an
amount, date, note, type, or status on a transaction that keeps its archived
account or category, but cannot choose an archived account or category for a
new or different association, and archived resources never become selectable
for other transactions.

**Alternatives considered**:

- Freeze all financial fields once an account or category is archived: rejected
  because users could not correct genuine mistakes.
- Force any edit to move the transaction to an active account: rejected because
  it silently rewrites history and removes the user's ability to keep it.
- Permit selecting any archived resource the user owns: rejected because it
  reintroduces archived choices in new entry.

## Decision: Account and category archiving never changes transaction state

**Rationale**: Account and category lifecycle belongs to those features, and
their archive actions already work regardless of associations. This feature
adds no blocking rule: a pending transaction on a resource that is being
archived stays pending, keeps contributing nothing to balances, and remains
visible and editable while keeping that archived association. Blocking archive
or auto-settling pending transactions would both surprise users and invent
cross-feature behavior that the account and category specifications do not
define.

**Alternatives considered**:

- Block archiving until pending transactions are resolved: rejected because it
  adds an unrequested constraint to the account and category features.
- Auto-mark pending transactions effective on archive: rejected because it
  changes balances the user did not ask to change.
- Auto-remove pending transactions on archive: rejected because it destroys user
  data and hides intent.

## Decision: Embed account and category summaries in the transaction resource

**Rationale**: History, detail, filters, and reports must stay understandable
when a referenced account or category has been archived. Embedding a minimal
summary (identifier, name, lifecycle status, and visual identifiers) keeps every
response self-explanatory and avoids an extra request per row.

**Alternatives considered**:

- Return only `financial_account_id` and `category_id`: rejected because the
  frontend would need extra lookups and archived resources may not appear in
  active lists.
- Persist denormalized name snapshots on the transaction: rejected because the
  account and category records already preserve their own history and renaming
  behavior belongs to those features.

## Decision: Server-side pagination with newest-first ordering

**Rationale**: The clarified scale rule is a history of 5,000 transactions per
user with progressive loading in batches of at most 50 and a visible count of
matching transactions. Server-side pagination with a total count delivers that
directly and keeps the response size bounded.

**Alternatives considered**:

- Loading the full history: rejected because a 5,000-row payload contradicts the
  performance criterion and the specified batch size.
- Client-side pagination over a full download: rejected for the same reason.
- Cursor pagination without a total: rejected because the matching count is
  required by the spec.

## Decision: Search text covers description and notes; structure comes from filters

**Rationale**: The clarified rule is that search matches description and note,
while type, account, category, status, and dates are selected as filters.
Matching ignores case and accents, consistent with the normalization already
used for account and category names, so users do not have to reproduce exact
capitalization or accents.

**Alternatives considered**:

- Searching category and account names as text: rejected because it duplicates
  structured filters and produces surprising matches.
- Searching notes only: rejected because recall usually starts from the
  description.
- Case-sensitive search: rejected as user-hostile for Portuguese text.

## Decision: Store a normalized search text column

**Rationale**: Accent- and case-insensitive matching must behave identically in
the MySQL test environment and the SQLite development default, and must not
depend on engine-specific collations. A normalized `search_text` column, built
from the lowercased, accent-stripped description and notes, mirrors the
`normalized_name` approach already used by accounts and categories and keeps
search deterministic and indexable.

**Alternatives considered**:

- Relying on the database collation: rejected because SQLite and MySQL differ,
  so tests and development would not agree.
- A database extension such as `unaccent`: rejected because it needs engine
  changes and adds a dependency.
- Matching raw columns with lower-cased input: rejected because accents would
  still break Portuguese searches.

## Decision: Freeze the forward-compatible history flags on first association

**Rationale**: The financial-account feature locks the initial balance once
`has_financial_movements` is true, and the category feature locks classification
once `has_financial_transactions` is true. Real transaction records must now set
those flags so the promised locks take effect with production data instead of
test fixtures. The flags stay true even if the transaction is later removed,
because the historical association still exists.

**Alternatives considered**:

- Leaving the flags untouched: rejected because account and category locks
  would remain inert for real users.
- Clearing a flag when the last transaction is removed: rejected because the
  historical association and any prior reporting cannot be undone safely.
- Replacing the flags with live counts: rejected as an unrequested schema
  migration of completed features.

## Decision: Normalize dates to a stored date value with a fixed supported range

**Rationale**: Transaction dates drive ordering, filtering, and the
future-dated rule. Accepting ISO `YYYY-MM-DD` and Brazilian `DD/MM/YYYY` input,
normalizing to a stored date, and restricting values to 1900-01-01 through
2100-12-31 rejects typos while covering every plausible past entry and planned
future movement.

**Alternatives considered**:

- Storing a full timestamp: rejected because the financial event is a calendar
  date and a time component would create timezone ambiguity in ordering.
- Unlimited dates: rejected because typing mistakes silently create absurd
  history.
- ISO input only: rejected because the product targets Brazilian users who
  enter `DD/MM/YYYY`.

## Decision: Reuse the existing API envelope and lifecycle route conventions

**Rationale**: Transactions reuse the versioned `/api/v1` prefix, the
authenticated `auth:sanctum` plus `session.lifetime` middleware group, the
`{ "data": ... }` response envelope, and POST lifecycle sub-routes like the
account and category features (`POST /transactions/{id}/remove`,
`POST /transactions/{id}/restore`). Consistent shapes keep the frontend service
layer predictable.

**Alternatives considered**:

- `DELETE /transactions/{id}`: rejected because removal is not a permanent
  delete and the response must describe a retained record.
- Per-verb controllers with different envelopes: rejected as inconsistent with
  the existing contract.
- Adding pagination metadata to every endpoint: rejected because only the
  history endpoint is paginated.
