# Data Model: Account Transfers

## Transfer

| Field | Type | Rules |
|---|---|---|
| id | integer | Generated public identifier. |
| user_id | integer | Authenticated owner; never accepted from input. |
| source_financial_account_id | integer | Required active owned account for new/replaced association; differs from destination. |
| destination_financial_account_id | integer | Required active owned account for new/replaced association; differs from source. |
| amount_centavos | integer | Positive BRL centavos, 1 through 99,999,999,999. |
| currency_code | string(3) | Always BRL. |
| transfer_date | date | Required; 1900-01-01 through 2100-12-31. |
| status | enum | pending or effective; see lifecycle. |
| description | string(200), nullable | Optional, trimmed; blank becomes null. |
| notes | text, nullable | Optional, trimmed, max 1,000 user-visible characters; blank becomes null. |
| search_text | string(1200) | Derived lowercased accent-stripped description and notes. |
| removed_at | timestamp, nullable | Set on Remove; cleared on Restore. |
| timestamps | timestamps | Standard creation/update context. |

## Transfer Mutation Request

| Field | Type | Rules |
|---|---|---|
| user_id | integer | Scopes key to owner. |
| idempotency_key | string(255) | Required for create, update, remove, restore. |
| operation | enum/string | create, update, remove, restore. |
| request_hash | string(64) | Operation, target, normalized-payload fingerprint. |
| transfer_id | integer, nullable | Target/result transfer. |
| response_status / response_body | integer / JSON | Completed response replayed for exact retry. |

## Financial History Entry

Read projection in existing financial/transaction history. Its movement_kind is
income, expense, or transfer. Transfer entry includes source/destination account
summaries, no category, and non-color label such as Transfer: Source →
Destination. It does not contribute to income/expense aggregates.

Income and expense entries also expose optional `recurrence_source` (originating
recurring rule identifier plus scheduled date), added by the Recurring
Transactions feature. Transfers never carry a recurrence source, and this
additive nullable field does not change existing history behavior.

The projection is served by `GET /financial-history` and also publishes
`meta.totals` (`income_centavos`, `expense_centavos`,
`financial_result_centavos`) computed from effective, non-removed income and
expense transactions only. Feature 011 made these totals period-aware: when the
request carries `from` and/or `to`, only transactions dated inside that inclusive
range contribute, so the totals always describe the listed period; a request
without a period keeps the original all-time totals.

## Aggregate Boundaries

FinancialHistoryService totals classify only income and expense entries; transfer
entries are excluded from income, expense, and financial-result totals.
FinancialAccountService applies the two account effects and derives net worth
from the combined owned-account balances, so a transfer changes each side but
never the combined total. Any future report must use the same movement-kind
classification before it is exposed.

## Account Effect

| Side | Effective non-removed transfer effect |
|---|---|
| Source account | Negative amount_centavos |
| Destination account | Positive amount_centavos |
| Pending or removed transfer | Zero for both accounts |

For every account in old or new transfer state:

    proposed_balance = current_balance - before_effect + after_effect

Lock all distinct affected accounts in ascending identifier order in one
transaction. Before completion each proposed destination remains within account
balance range and source in new effective transfer has proposed balance at least
zero. Failure rolls back transfer, idempotency, and all balance changes.

## Status and Removal Lifecycle

| From | Action | To | Balance effect |
|---|---|---|---|
| pending | Mark effective | effective | Apply debit/credit once after source-funds validation. |
| effective | Mark pending | pending | Reverse both effects once. |
| any active | Remove | removed | Reverse both effects once if effective. |
| removed | Restore as pending | pending | No effect. |
| removed | Restore as effective | effective | Apply both effects once after source validation. |
| removed | Edit | removed | Reject; restore first. |

Repeat removal of removed transfer and restore of active transfer are conflicts.

## Date Rules

- New past/current transfers default effective.
- New/restore future transfers default pending; requested effective future state
  is rejected.
- Date edit never silently changes status.
- Effective transfer retimed future stays effective and returns notice; user may
  explicitly mark pending.

## Relationships and Historical Rules

- User has many transfers.
- Transfer has one source and one destination financial account, owned by same
  user.
- Financial account has source and destination transfer histories.
- First association sets financial_accounts.has_financial_movements true
  permanently, preserving initial-balance lock.
- Archived account remains embedded historical summary. Archiving does not
  change transfer lifecycle or balance effect.

## Filtering and Ordering

| Input | Meaning |
|---|---|
| from, to | Inclusive transfer-date range. |
| source_financial_account_id | Owner active or archived source account. |
| destination_financial_account_id | Owner active or archived destination account. |
| status | pending or effective. |
| q | Normalized description/notes search. |
| view | active default or removed. |
| page, per_page | Default/max page size 50. |

All filters combine with AND semantics. Order: date descending, then identifier.
