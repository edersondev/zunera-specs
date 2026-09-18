# Research: Monthly Budgets

## Financial truth and full-month boundary

**Decision**: Derive financial values from owned transaction rows at read-time.
Realized includes only non-removed, effective, expense transactions in full
selected calendar month. Use America/Sao_Paulo first/last day, not dashboard's
current-month-through-today period.

**Rationale**: Transaction counting and Dashboard distribution already define
effective, non-removed truth. Effective future-dated corrections belong to their
stored month.

**Alternatives considered**: Persist realized amounts (stale after correction);
client/paginated aggregation (incomplete); dashboard current period (omits dates).

## Expected and projected scope

**Decision**: Expected equals owned non-removed pending expense transactions in
matching category/current-or-future budget month. Generated pending recurrence
transaction qualifies; recurrence definition alone does not. Ended months omit
expected/projected values.

**Rationale**: Matches clarification and prevents Dashboard's 30-day recurrence
forecast becoming competing Budget calculation.

**Alternatives considered**: Include recurrence definitions; merge expected into
realized; show projections in historical month.

## Historical category identity

**Decision**: Persist each plan's category name, classification, origin, color,
and icon snapshot. Keep category relation for calculations/lifecycle.

**Rationale**: Current Category data can rename/change visual identity; snapshots
preserve historic plan meaning after change/archive.

**Alternatives considered**: Render current category (rewrites history);
name-only snapshot (loses visual identity); duplicate category records.

## Archived plan lifecycle

**Decision**: Linked archived category makes plan read-only while keeping it in
calculations. Category restoration enables mutation; snapshot remains immutable.

**Rationale**: Matches clarified behavior and established archived-association
rules without allowing new unavailable association.

**Alternatives considered**: Edit while archived; automatically remove plan.

## Category classification after planning

**Decision**: First budget-plan association locks category classification as
expense, even if category has no transaction. Removing a plan never unlocks it.

**Rationale**: An active category must not silently become income while retaining
expense planning history and derived expense rules.

**Alternatives considered**: Leave conflicting plan read-only; automatically
remove plans after reclassification.

## Copy safety

**Decision**: Copy only plan inputs in one transaction. Destination must differ,
be empty, and belong to owner; every source category must be active. Any archived
source plan blocks whole copy. Destination snapshots current active category.

**Rationale**: All-or-nothing prevents partial/unusable plan and uniqueness race.

**Alternatives considered**: Merge/rewrite destination; omit archive; clone it
read-only.

## API, frontend, and performance

**Decision**: Protected resource routes return normal `200` `budget: null` for
empty selected month; Form Requests/DTOs validate month/plan/copy; Resources
stabilize shapes. Frontend uses Axios service + feature Pinia setup store and
server-derived values with textual Element Plus progress. Reuse 10,000 movement/
2-second dashboard baseline; add index only after measurement.

**Rationale**: Empty month is expected, controllers remain thin, and existing
Vue/Pinia dashboard patterns isolate async state. Server aggregation avoids
client financial truth and meets scale baseline.

**Alternatives considered**: `404` no budget; monolithic browser calculation;
new chart package; speculative cache/index.
