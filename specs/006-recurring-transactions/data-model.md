# Data Model: Recurring Transactions

## Recurring Transaction

| Field | Type | Rules |
|---|---|---|
| id | integer | Generated identifier. |
| user_id | integer | Authenticated owner; never client input. |
| financial_account_id | integer | Required owned active account on create/reassignment. |
| category_id | integer | Required active available category matching type. |
| type | enum | `income` or `expense`. |
| amount_centavos | integer | 1–99,999,999,999; exact BRL cents. |
| currency_code | string(3) | `BRL`. |
| description / notes | string(200) / nullable text | Trimmed nonblank / optional max 1,000 visible chars. |
| frequency | enum | `weekly`, `monthly`, `yearly`. |
| start_date / end_date | date / nullable date | Valid range; end inclusive and not before start. |
| state | enum | `active`, `paused`, `ended`. |
| paused_reason | nullable enum | `user` or `association_archived`. |
| eligibility_starts_on | date | Later of start and creation business date. |
| schedule_cursor | date | Newest schedule date already evaluated or intentionally skipped. Seeded to the eligibility start when the rule is created, advanced by the due processor for active rules, and set to the resume business date when a paused rule resumes, so paused dates are never evaluated. |
| ended_at / timestamps | timestamps | Terminal and audit context. |

Indexes: owner/state/scheduling; owner/account; owner/category. List ordering is
rules with a next expected occurrence first in ascending date order, then rules
without one by identifier, with stable pages of at most 50.

`next_expected_occurrence` is present only for an active rule whose start, end,
and calendar rules still allow a future schedule date. Paused and ended rules
report no next expected occurrence until the owner resumes them.

## Generated Transaction Extension

Existing `Transaction` gains nullable `recurring_transaction_id` and
`recurrence_scheduled_date`. Both are required together for a generated
occurrence and null for ordinary entry. Unique non-null pair
`(recurring_transaction_id, recurrence_scheduled_date)` guarantees one
occurrence per rule/date; removed occurrence retains pair and is never
regenerated. Existing fields are immutable source snapshot; generated status
begins pending and balance effect stays zero until ordinary transaction action
makes it effective.

Rule detail exposes only the generated-occurrence count. Generated ordinary
transactions are listed through an owner-scoped paginated occurrence collection
(at most 50 per page), so a long-running rule never returns an unbounded detail
payload and each summary can open its existing transaction.

## Recurrence Mutation Request

Owner-scoped idempotency record: `idempotency_key`, operation (`create`,
`update`, `pause`, `resume`, `end`), normalized request hash, target/result rule,
and completed status/body. Exact retry replays; altered reuse is conflict.

## Lifecycle and Processing

| From | Event | To | Effect |
|---|---|---|---|
| active | Owner pauses | paused | Skip future dates; reason `user`; cursor stays. |
| active | Account/category archives | paused | Skip future dates; reason `association_archived`; cursor stays. |
| paused | Repair and resume | active | Future dates only; cursor set to resume business date so paused dates remain skipped. |
| active/paused | Owner ends or end date passes | ended | Terminal; history retained. |

For active rules processor evaluates dates from `schedule_cursor` through current
Brazil business date and inclusive end, never earlier than
`eligibility_starts_on`. Each missing date creates one pending source
transaction, then the cursor advances to the business date already evaluated.
It then ends rules whose end date has passed. Restored association stays paused
until owner resumes, and resume never backfills the paused window.

## Relationships and Historical Rules

- User has recurring transactions and transactions.
- Recurrence has one account, one matching category, many generated transactions.
- Generated transaction optionally belongs to recurrence and preserves source
  financial snapshot after later rule edits.
- Account/category archive pauses recurrence only; no existing occurrence,
  transaction state, or balance effect changes.
- Individual correction/removal/restore changes one transaction, never rule or
  sibling occurrence.
