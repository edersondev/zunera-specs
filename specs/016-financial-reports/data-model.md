# Data Model: Financial Reports

**Date**: 2026-09-26 | **Spec**: [spec.md](spec.md) | **Research**: [research.md](research.md)

Reports are read-only projections. No report balance, transaction copy, cache table, or report-persistence migration is planned. Existing Transactions, Transfers, Categories, Financial Accounts, Credit Card purchases/installments/statements/credit events/payments, and Recurring occurrences remain source records. Financial Goal activity is excluded.

## Report scope

| Field | Meaning / validation |
|---|---|
| `user_id` | Authenticated owner; applied to every source query. No separate workspace table exists today. |
| `preset` | `current_month`, `previous_month`, `historical_month`, `current_year`, `previous_year`, or `custom`. Four calendar shortcuts plus custom remain available; `historical_month` supports older completed months. “Last completed month” is the user-facing meaning of `previous_month`, with no duplicate preset or extra response value. |
| `month` | `YYYY-MM` only for `historical_month`; identifies a completed month before the current Sao Paulo business month. Required for that preset and rejected for every other preset. |
| `from`, `to` | Inclusive business dates in ISO calendar format. Both required for custom and rejected for every other preset; reject reversed, malformed, unsupported, or out-of-domain ranges. Server resolves calendar-choice boundaries. |
| `account_id` | Optional owned financial account, including an archived account with valid history; never a credit-card ID. |
| `category_id` | Optional owned income or expense category, including archived historical category. |
| `transaction_type` | Optional `income` or `expense` filter; transfers and settlements remain separate movement kinds. |
| `is_filtered` | True if any account/category/type filter is active; period selection alone does not mark financial scope as filtered. |

The validated scope is immutable during one report read. An inaccessible account/category is rejected without leaking its label. A valid but incompatible category/type combination yields a zero result and empty-state reason. Every overview section and detail request returns the resolved scope so the frontend can detect stale responses.

Each overview and contribution-detail result also carries an opaque `source_revision` for its resolved scope. It represents authoritative source identities, financially relevant states/effects, and report-visible identifying or explanatory information used by the report, including current and comparison periods and separate account movements. A changed contributor description or displayed category/account identity changes the revision even when the financial effect and total stay unchanged. It is read-only metadata, not a persisted report record or monetary input. Within one detail response, source rows, all-record total, and revision come from one coherent source state. A later page may represent a newer state; its changed revision must be detected before those rows are combined with earlier pages.

The delivered read calculates the revision from the resolved owner and scope, ordered recognized contribution rows for both periods, applicable account movement rows, and selected account/category labels. Overview and detail each use a database transaction so their independently queried parts see one repeatable-read snapshot in MySQL. Two separate requests can still see different revisions. The browser accepts their pair only when revisions and applied scopes agree; it retries the overview and first detail page up to three times, then offers a retry if sources continue changing.

## Reporting and comparison periods

| Field | Meaning |
|---|---|
| `preset` | Selected period choice. |
| `from`, `to` | Actual inclusive first/last business dates. |
| `day_count` | Inclusive number of days. |
| `comparison_from`, `comparison_to`, `comparison_day_count` | Server-derived prior range and length. |

Current presets end at current America/Sao_Paulo business date. Previous presets and `historical_month` cover complete calendar units; `historical_month` requires a completed month before the current business month. Completed month/year comparison, including `historical_month`, uses the preceding complete calendar month/year. Current month/year comparison uses matching prior calendar dates, capped for shorter month or mapped to February 28 for leap-day end. Custom comparison uses the immediately preceding equal-day range even when custom dates happen to span a full calendar month. Calendar comparisons may have unequal day counts; then percentage change is unavailable.

## Recognized financial contribution

Virtual read model used by all income/expense aggregates and detail:

| Field | Meaning |
|---|---|
| `source_kind`, `source_id` | `ordinary_transaction`, `card_installment`, or `card_credit_adjustment`; stable source identity. |
| `related_purchase_id`, `related_statement_id`, `related_installment_id`, `related_credit_event_id` | Card traceability where applicable; null otherwise. Credit-event plus installment identifies each allocated adjustment. |
| `recognized_date` | Ordinary transaction date or associated card statement closing date. Credit adjustment uses affected installment's closing date, not credit-event recording date. |
| `classification` | `income` or `expense`; card installments/adjustments remain expense. |
| `category_id` | Existing authoritative category. Preserve archived identity on historical read. |
| `financial_account_id` | Ordinary transaction account; null for card installment and its credit adjustment. |
| `signed_amount_centavos` | Positive ordinary/card principal contribution; negative card credit adjustment against source installment. Exact BRL integer. |
| `description`, `source_status` | Safe explanatory label and current source state for detail, never a reporting copy. |

Ordinary inclusion requires owner, financially effective state, nonremoved record, and `transaction_date` inside period. An existing effective future-dated ordinary transaction follows Spec 004 and is included when its date falls in a selected future range. Card inclusion requires owner, associated statement closing date inside period and strictly before current business date, as in the existing card projection. Open-statement installments remain absent. A generated recurring transaction or card purchase uses these same rules, with no second contribution from its recurrence definition or generation event.

For each card purchase, accepted credit events reduce recognized principal in original installment sequence by centavo, irrespective of payment state. The allocation is a read-model correction to the existing credit-event recognition rule, not a new cash or obligation movement. One event may yield several signed adjustment contributions, each linked to event and affected installment, dated in the installment period. Sum of adjustment magnitudes equals accepted credit-event amount; each installment's recognized net is between zero and its original amount. Credit applications against statements and remaining card credit still follow existing obligation logic and never produce income/expense contributions.

## Account movement

Virtual account-activity source distinct from recognized income/expense:

| Field | Meaning |
|---|---|
| `source_kind`, `source_id` | `transfer` or `card_statement_payment`. |
| `account_id` | Involved financial account. One effective transfer yields outgoing movement for source and incoming movement for destination; one effective payment yields settlement for paying account. |
| `movement_date` | Authoritative effective transfer/payment date in selected period. |
| `kind` | `transfer_in`, `transfer_out`, or `card_settlement`. |
| `amount_centavos` | Positive magnitude for each separate movement figure. |

Pending or removed movement is excluded. Transfer directions appear only for the respective account. The two sides of one transfer do not enter workspace income/expense. Card settlement affects account cash balance and card obligation under existing rules, but not Reports expense. Goal allocation/release never enters this source.

When `transaction_type` is `income` or `expense`, transfer and settlement movement figures are zero because those movements do not match either selected type. Without a type filter, they appear separately. Category filters likewise exclude uncategorized transfer/settlement movements. This keeps account sections aligned with active filters while never reclassifying the movement.

Account analysis includes accounts with matching recognized activity or effective movement, including archived accounts. A specifically selected owned account remains visible with zero figures when it has no matching activity, so the filtered empty state has clear account context. Unselected zero-activity accounts need not fill the report.

## Derived projections

| Projection | Derivation and invariant |
|---|---|
| Summary | Sum income contributions; sum signed expense contributions; result = income − expenses. |
| Evolution bucket | Group contributions by the established daily/weekly/monthly intervals; buckets cover period without gap/overlap; interval sums equal summary. |
| Expense category | Sum signed expense contributions by category; sum category totals equals summary expenses under identical scope. Share = category / expenses × 100 when denominator positive, otherwise unavailable. |
| Income category | Sum income contributions by category, separate from expense categories; sum equals summary income. |
| Account row | Owner's account identity and sum of attributed ordinary income/direct expense; net financial flow = income − direct expense; separate incoming/outgoing transfer and settlement magnitudes. Card spending is deliberately unassigned to financial accounts. |
| Unattributed card expenses | Realized net card-installment expense included in workspace-wide expense but in no financial-account row. With no account filter, account direct-expense totals plus this amount equal summary expenses under the same category/type scope; with an account filter it is zero. |
| Comparison | Recompute same projection for prior range under identical filters; signed absolute difference = current − previous. Percentage is null when day counts differ, previous value nonpositive, or result changes sign. |
| Detail total and page | Reuse exact recognized-contribution or account-movement scope for selected metric. All-record total equals overview metric; rows expose metric-relative signed contribution and stable source identity. For expense metrics, principal is positive and refund adjustment negative. For result/net-flow metrics, expenses are negative and expense-reducing adjustments positive. Pagination never changes all-record total. |

Every amount uses integer centavos until locale formatting at presentation. Percentage rounding is display-only and cannot change reconciliation. Summary, category, interval, account, and detail reads share one server-derived scope. Archived records remain readable; no current balances are calculated or stored by Reports.

The authoritative Transaction and card purchase rules currently require an eligible category for their reportable income and expense records. If a valid uncategorized source becomes possible under those rules, category projections and comparisons use a null category identity for an identifiable uncategorized row and reconcile with the matching summary; its category contribution detail is selected without a category ID. Reports must not silently discard that source or invent a category record for the current required-category model. An omitted category metric ID remains invalid while no valid uncategorized source exists.

## Read lifecycle and error states

Reports have no write lifecycle. The read flow is: authenticate → validate period/filters → resolve owner-scoped identities and effective dates → read contributions/movements from one coherent source state per response → aggregate current/prior scopes and source revision → return labelled results. Verify overview and detail read boundaries with concurrent-change regressions; use a consistent read snapshot wherever independently read values can mix states. A source correction or state transition changes the next read, with no report cache to invalidate. Detail is a later read and may observe a newer state: compare its source revision to the overview's, then automatically refresh both under the same scope and explain the change if they differ. Compare later detail pages before combining their rows with earlier pages. Show overview and detail together only when revisions agree; bound retries and offer a recoverable retry state if sources keep changing. The all-record total still reconciles the selected metric, but equal totals do not prove equal contributors. Invalid range/filter produces a validation error; foreign identity is undisclosed; empty current/prior scope gets a typed empty reason. An independently failed section returns an unavailable state and null content, while reliable sections retain the same scope and stay usable; an available section with no records returns its empty value and available state.

## Scale and indexing review

Existing source-table owner, state, date, category, account, statement, and purchase indexes must be reviewed against 10,000-record/100-category/50-account fixtures. Add source-table index migrations only if query plans demonstrate need during implementation. Detail uses bounded, stable pages and a separate all-record aggregate. Do not add a reporting table or fetch all source records into the browser.
