# Data Model: Monthly Budgets

## Persistence and ownership

Budget records are planning data, never transactions. All money is integer
centavos with `currency_code: BRL`. Realized, expected, projected, available,
utilization, status, and totals are derived response values, never stored truth.

### MonthlyBudget

| Field | Rules |
|---|---|
| `id` | Stable internal identifier |
| `user_id` | Required owner; every access constrains it |
| `budget_year` | 1900–2100 |
| `budget_month` | 1–12 |
| timestamps | Planning-definition audit only |

Unique (`user_id`, `budget_year`, `budget_month`); indexed owner/month. Empty
budget valid; period immutable; no delete-month action in v1.

### BudgetCategoryPlan

| Field | Rules |
|---|---|
| `id` | Stable internal identifier |
| `monthly_budget_id` | Required parent |
| `category_id` | Required linked eligible expense category |
| `planned_amount_centavos` | Integer 1–99,999,999,999 |
| `currency_code` | Constant `BRL` |
| `category_name_snapshot` | Required captured identity |
| `category_classification_snapshot` | Required `expense` |
| `category_origin_snapshot` | Required system/personal origin |
| `category_color_snapshot` / `category_icon_snapshot` | Nullable captured visual identity |
| timestamps | Planning-definition audit only |

Unique (`monthly_budget_id`, `category_id`). Snapshot never changes. New plan
requires active available expense category. Archived relation is read-only;
restoration reenables mutation without replacing snapshot. Any plan association
locks linked category classification as expense, even before a transaction;
removing last plan does not unlock it.

## Existing authoritative sources

| Source | Budget rule |
|---|---|
| `Transaction` | Realized: owned active/effective expense/category/month. Expected: owned active/pending expense/category/current-or-future month. |
| `Transfer` | Never joins spending. |
| `Category` | Active available expense valid for new plan; archive controls mutation; any plan locks classification as expense. |
| `RecurringTransaction` | Never direct spending; only generated pending transaction may be expected. |
| `FinancialAccount` | Never receives Budget write. |

## Derived read model

### CategoryPlanProjection

| Field | Rule |
|---|---|
| `planned` | Persisted plan amount |
| `realized` | Qualifying effective sum |
| `available` | planned − realized; may negative |
| `utilization_percent` | realized ÷ planned × 100 |
| `status` | within / approaching / reached / exceeded, 80/100 thresholds |
| `expected` | Qualifying pending sum; omitted/null past month |
| `projected_spending` | realized + expected |
| `projected_available` | planned − projected spending |
| `projected_status` | Projection only; never actual status |
| `is_read_only` | Current linked category archived |

### MonthlyBudgetProjection

| Field | Rule |
|---|---|
| `period` | Selected full calendar month |
| `budget` | `null` for normal no-budget state |
| `total_planned` | Sum plan amounts |
| `budgeted_realized` | Sum plan realized |
| `actual_available` | total planned − budgeted realized |
| `overall_utilization_percent` | budgeted realized ÷ total planned; `null` if no plans |
| `overall_status` | status or `not_applicable` if no plans |
| `unbudgeted_expenses` | Effective monthly expenses without plan category |
| `total_expenses` | budgeted realized + unbudgeted |

Source corrections (state, removal/restoration, amount, type, category, date)
recalculate old/new matching period/category next read. Income/transfers stay
excluded. Removing a plan makes qualifying category spending unbudgeted.
