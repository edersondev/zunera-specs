# Research: Financial Dashboard

## Authoritative financial-state boundary

**Decision**: Derive projections server-side from owned source records. A
transaction counts only when non-removed and `effective`; income adds its positive
centavo amount and expense subtracts it. Current balance is active owned accounts
only. Transfers never enter realized totals, distribution, or evolution.

**Rationale**: Existing history is paginated and its totals are all-time, so client
aggregation is incorrect for selected periods and cannot meet 10,000-movement goal.

**Alternatives considered**: Reuse history totals (wrong date scope); persist
dashboard totals (diverge after corrections); count transfers (contradicts rules).

## Independent projections

**Decision**: Expose authenticated GET projections for summary, accounts, expense
distribution, evolution, recent activity, and upcoming activity.

**Rationale**: Frontend slices load/retry independently; one failed section does
not hide reliable content.

**Alternatives considered**: One large endpoint; client assembly from paginated
lists.

## Period, evolution, and history

**Decision**: Validate current-month, previous-month, and custom inclusive periods
in DTO/Form Request. Default uses America/Sao_Paulo business date. Use daily
buckets through 31 days, weekly through 93, monthly beyond; mark partial boundary
buckets and never compare them as complete.

**Rationale**: Shared validation makes section dates and chart acceptance tests
deterministic. Effective transaction re-dated future remains effective if its
existing lifecycle allows it, but reporting uses stored transaction date.

## Upcoming activity

**Decision**: In next 30 calendar days show active non-removed future pending
transactions and every eligible future date of active recurrence. Suppress a rule
projection when generated pending transaction already represents same rule/date.

**Rationale**: Rule itself never moves money; recurrence list supplies only one
next date although weekly rules can have several within horizon.

**Alternatives considered**: Rule-only/one-next-date list; treating planned as
realized; putting past/current pending into expected instead of Recent Activity.

## Historical integrity and accessible charts

**Decision**: Active accounts supply current total; valid historical records keep
archived account/category snapshots/status. Removed rows never appear. Use local
semantic SVG/data-led visualizations plus legend, values, and table/text alternative.

**Rationale**: Reuses lifecycle rules and meets Design Foundation without
unapproved chart package. Category palette means category identity only;
financial direction uses sign/label/icon plus semantic token.

## Ownership and performance

**Decision**: Sanctum-protect every route, derive owner from user, validate input,
use owner/date/state constrained aggregates and bounded activity reads. Validate
10,000 representative movements at ≤2 seconds.

**Alternatives considered**: Arbitrary input; load/filter every row; introduce
cache or persisted report table before measurement.
