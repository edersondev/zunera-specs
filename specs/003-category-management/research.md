# Research: Category Management

## Decision: Persist system defaults and personal categories together

**Rationale**: Persisted system rows provide stable identifiers, explicit
origin, deterministic availability in fresh installations, and a future-safe
target for transaction associations. A personal row has exactly one owner;
system rows have no user owner and cannot be changed by users.

**Alternatives considered**:

- Static application constants: rejected because defaults would not have stable
  resource identity or a direct future transaction association.
- Per-user copied defaults: rejected because it duplicates shared system data,
  makes defaults look user-owned, and complicates updates.

## Decision: Combine defaults with only the signed-in user's personal rows

**Rationale**: The active category list must show all defaults plus personal
active categories, while archived results must contain only the signed-in
user's archived personal categories. This directly enforces ownership and keeps
new-transaction choices free of archived rows.

**Alternatives considered**:

- Return all users' categories: rejected for privacy and ownership violations.
- Archive system defaults per user: rejected by clarification; defaults are
  always visible and read-only.

## Decision: Use income and expense classifications only

**Rationale**: These classifications cover the specified default set and are
the only confirmed scope. Restricting them prevents ambiguous category behavior
until a future feature defines another classification's rules.

**Alternatives considered**:

- Transfer classification: rejected as out of scope because transfer behavior
  has not been specified.
- User-defined classifications: rejected because it weakens validation and
  reporting semantics.

## Decision: Lock classification after first financial transaction association

**Rationale**: Classification determines the financial meaning of historic
transactions. Once used, changing it would reclassify history. The existing
financial-account domain already uses a forward-compatible history flag, so
categories use `has_financial_transactions` until transaction records exist.

**Alternatives considered**:

- Reclassify history: rejected because it changes historical information.
- Version a category per association: rejected because it expands transaction
history scope before transactions exist.

## Decision: Enforce normalized active-name uniqueness with a service check for defaults

**Rationale**: Personal active names must be unique per owner and
classification after whitespace, case, and accent normalization. Persistence
prevents concurrent personal duplicates. Service validation also rejects a
personal name matching a system default, since a user-only uniqueness rule
cannot express that cross-origin conflict.

**Alternatives considered**:

- Raw display-name comparison: rejected because visually equivalent variants
  would create ambiguous choices.
- Permit default-name duplicates: rejected by the specification's ambiguity
  rule.

## Decision: Use active/archived lifecycle, never deletion

**Rationale**: Archive removes a personal category from normal new-transaction
selection while preserving historic associations. Restore can fail only for a
valid state or normalized active-name conflict.

**Alternatives considered**:

- Permanent deletion: rejected because it can break historical information.
- Separate inactive and archived states: rejected by clarification; they are
  user-facing synonyms for one archived state.

## Decision: Use chart-palette visual identifiers, separate from semantic colors

**Rationale**: The Design Foundation reserves green/red and status colors for
income, expense, success, warning, and error. Its categorical chart palette is
expressly intended for category distinction. Color remains supplemental to
name, classification, origin, and state labels.

**Alternatives considered**:

- Arbitrary user-entered colors: rejected because they can violate theme
  contrast and semantic-color rules.
- Financial semantic colors for categories: rejected because visual identity
  must not override financial meaning.

## Decision: Reuse existing resource, service, and UI boundaries

**Rationale**: Current financial-account code already provides protected v1
resource routing, Form Requests, DTOs, services, API Resources, typed conflicts,
Axios services, setup-style Pinia stores, small focused Vue components, and
isolated Playwright journeys. Current Laravel documentation supports Form
Request policy authorization and resource responses; current Vue documentation
supports composition through focused stateful functions; current Element Plus
documentation supports form-rule validation and confirmation dialogs.

**Alternatives considered**:

- Introduce new packages or a global category UI framework: rejected by the
  constitution and existing feature conventions.
- Put API calls or domain rules in Vue components: rejected by the constitution
  and Design Foundation component boundaries.
