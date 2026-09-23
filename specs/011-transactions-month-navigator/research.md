# Research: Transactions Month Navigator

## Where the period-scoped totals belong

**Decision**: Scope `meta.totals` inside the existing
`GET /api/v1/financial-history` request by passing the already-built filter data
to `FinancialHistoryService::totals`, applying `from` and `to` to
`transactions.transaction_date`.

**Rationale**: The list and the totals are returned by the same request, so one
filter object guarantees they can never disagree; no new endpoint, no second
round trip, and no additional parameter for the frontend to keep in sync.
Omitting the period preserves the current all-time response.

**Alternatives considered**: A separate totals endpoint (extra round trip and a
second source of truth for the same screen); a `month` query parameter (a second
period vocabulary next to the existing `from`/`to` and the existing date-range
filter); recomputing totals on the client (would duplicate reporting rules and
break pagination accuracy).

## Which fields narrow the totals

**Decision**: Only the period narrows totals. Type, status, account, category, and
search keep their existing list-only effect, and realized income/expense rules
(effective, non-removed, transfers and recognized credit-card expenses excluded)
stay exactly as documented today.

**Rationale**: This keeps the change additive and predictable: the cards describe
the displayed month, which is the requested outcome, without redefining what
"realized" means for the other criteria.

**Alternatives considered**: Applying every active criterion to the totals
(broader behavior change, would need new copy explaining why searching by text
changes income); scoping only on the client (impossible with pagination).

## Month control ownership

**Decision**: Promote `BudgetMonthNavigator` to a shared
`components/common/MonthNavigator.vue` with `month`, `loading`, and
`change-month` as its whole contract, and move the month helpers
(`businessMonth`, `formatMonth`, `shiftMonth`) plus new `monthBounds` and
`monthFromDate` into `utils/common/monthFormatters.js`, re-exported by the budgets
formatter module.

**Rationale**: Both pages need identical label formatting, year-boundary
arithmetic, and disabled-while-loading behavior. A single control removes drift
while keeping budget code paths untouched through re-exports.

**Alternatives considered**: Copying the budgets component into the transactions
folder (duplicate behavior and two places to fix later); making the budgets
component configurable with new props (couples budgets wording to transactions).

## Selected month state and address sync

**Decision**: The transactions view owns the selected month. It derives the
initial month from the address period when present and otherwise from the
business month, always requests the inclusive first-to-last day range together
with the other criteria, and writes the period back into the address with the
existing filter query sync.

**Rationale**: The address already carries `from`/`to`, so reload, back, and
shared links work without a new parameter, and the store keeps one filtered
request path.

**Alternatives considered**: A `month=YYYY-MM` parameter (duplicate period
representation to keep aligned with the date-range filter); keeping month state
only in memory (breaks reload and shared links).

## Custom range, criterion chip, and clearing

**Decision**: A custom range from the filters dialog overrides the month until an
arrow is clicked, which replaces it with the whole chosen month. The period
criterion appears only while the period differs from the navigator month, and
removing it or clearing filters returns to that month.

**Rationale**: The month navigator is the page's baseline scope, so it must not be
silently discarded, and the criteria strip must describe what the user added on
top of that baseline. The unfiltered empty state therefore stays reachable for a
month with no movements.

**Alternatives considered**: Treating the default month as an active criterion
(noisy chip on every load and a filtered-empty message for brand-new accounts);
allowing a custom range to disable the arrows (extra state to explain).

## Verification approach

**Decision**: Cover period totals with Laravel feature tests (in-range,
out-of-range, boundary days, empty month, pending/removed exclusion, and no-period
parity), cover the shared component, filter row, and view with Vitest, and extend
the transactions Playwright journey so its mock honors the requested period and
asserts list, totals, and address together.

**Rationale**: The risk lives at the boundary between the period filter and the
aggregate, plus the layout and criterion behavior of the filter row, so each
layer gets direct evidence.

**Alternatives considered**: Component-only coverage (would not catch a totals
regression); relying on manual checks (not repeatable evidence).
