# Data Model: Financial Dashboard

## Ownership and persistence

Dashboard has no persisted aggregate, migration, or independent financial entity.
Every projection is read-only and owner-scoped. Amounts remain integer
`*_centavos` with `currency_code: BRL`.

## Inputs

### ReportingPeriod

| Field | Rules |
|---|---|
| `preset` | `current_month`, `previous_month`, or `custom`; absent input defaults current month |
| `from` / `to` | Required together for `custom`; inclusive ISO dates; `from <= to`; existing financial date range applies |
| `interval` | Derived: daily ≤31 days; weekly ≤93; monthly otherwise |

### Source constraints

| Source | Counting / visibility rule |
|---|---|
| Transaction | Realized iff owned, `effective`, non-removed, date in period |
| Transfer | Never income/expense/result/distribution/evolution; present only in Recent Activity |
| Financial Account | Current total and overview only when owned and active |
| Category | Archived category remains attached to valid historical expense distribution |
| Recurring rule | Never balance effect; active eligible future dates project expected activity only |
| Generated occurrence | Existing transaction wins over matching recurrence/date projection |

Effective transactions whose stored date is edited into the future keep their
effective state and are reported in the period containing that stored date; a
newly recorded future-dated transaction stays pending under Transactions rules.

## Read projections

| Projection | Fields / relationship |
|---|---|
| `DashboardSummary` | current active total, period income, expense, result, currency, selected period |
| `AccountOverviewItem` | active account identity/type/status, balance, allocation share |
| `ExpenseDistributionItem` | category snapshot/id/status, total, share, descending rank |
| `EvolutionInterval` | inclusive start/end, partial marker, label, income/expense/result |
| `RecentActivityItem` | up to ten newest movement kind/id/status/date/amount/description/account(s)/category/recurrence source items; full history remains linked |
| `UpcomingActivityItem` | source kind, expected date, income/expense type, amount, account, category, description |

## State and correction effects

Dashboard has no state transition. Source changes affect next projection:
effective→pending/removal excludes realized amounts; restoration/effective
transition includes them; edit amount/type/date/account/category changes relevant
projections. Archiving account/category changes current inclusion only, not valid
historical interpretation.
