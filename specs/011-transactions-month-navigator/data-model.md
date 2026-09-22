# Data Model: Transactions Month Navigator

## Overview

No stored field, table, or migration changes. The feature adds period semantics to
an existing read projection and keeps the selected month in frontend view state
that is mirrored in the page address.

## Month Period (transient)

- **Shape**: `{ year, month, from, to }` derived for the selected calendar month,
  with `from` on the first day and `to` on the last day of that month.
- **Validation**: Month and year are positive integers; range boundaries are
  inclusive; the last day resolves month length, including leap-year February.
- **Lifecycle**: Created from the business month on first load, from the address
  period when a link or reload provides one, or from an arrow selection; a custom
  range replaces it until the next arrow selection.
- **Transport**: Sent as the existing `from` and `to` query parameters of
  `GET /api/v1/financial-history`.

## Financial History Totals (existing projection, extended semantics)

- **Shape**: `income_centavos`, `expense_centavos`, `financial_result_centavos`,
  `currency_code` inside the response `meta`.
- **Source**: Effective, non-removed transactions of type income or expense owned
  by the signed-in user.
- **Period rule**: When the request carries `from` and/or `to`, only transactions
  whose `transaction_date` falls inside the inclusive range contribute. Without a
  period, every owned movement contributes, preserving current behavior.
- **Exclusions**: Transfers and recognized credit-card expenses never contribute;
  pending and removed transactions never contribute.
- **Result rule**: `financial_result_centavos` is income minus expense for the same
  period, so a month without qualifying movements reports zero for all three
  values.
- **Invariants**: Totals and list share one validated filter object per request, so
  the cards can never describe a different period than the movements shown.

## Frontend View State

- **Selected month**: view-owned reactive month object that drives the navigator
  label, the outgoing period, and the criterion baseline.
- **Filters**: existing store filter object, now always carrying a period while the
  transactions page is open; cleared criteria keep that period.
- **Empty-state distinction**: the page is "filtered" only when criteria beyond
  the selected month are active, so a month without movements still shows the
  unfiltered empty state.

## Out of Scope

- No persisted user preference for the last viewed month.
- No change to budgets, dashboard presets, or removed-transactions data.
