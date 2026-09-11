# Data Model: Transaction Management

## Transaction

The recorded financial movement affecting exactly one of the signed-in user's
financial accounts.

| Field | Type | Rules |
|-------|------|-------|
| `id` | integer, primary key | Generated |
| `user_id` | integer, foreign key to `users` | Required, derived from the authenticated user, never accepted from input |
| `financial_account_id` | integer, foreign key to `financial_accounts` | Required, must belong to `user_id`, must be active for a new or changed association |
| `category_id` | integer, foreign key to `categories` | Required, must be available to `user_id` (own personal or any system default), must match the transaction type classification, must be active for a new or changed association |
| `type` | enum `TransactionType` | Required, one of `income`, `expense` |
| `status` | enum `TransactionStatus` | Required, one of `pending`, `effective`; defaults to `effective`, or to `pending` when `transaction_date` is in the future |
| `description` | string(200) | Required after trimming, must contain at least one non-whitespace character, maximum 200 user-visible characters |
| `notes` | text, nullable | Optional, maximum 1000 user-visible characters after trimming; empty input stored as `null` |
| `amount_centavos` | integer | Required, always positive, `1` to `99_999_999_999` (R$ 0.01 to R$ 999,999,999.99) |
| `currency_code` | string(3) | Always `BRL` in this feature |
| `transaction_date` | date | Required, `1900-01-01` through `2100-12-31`, accepted as ISO `YYYY-MM-DD` or Brazilian `DD/MM/YYYY` and normalized |
| `search_text` | string(1400) | Derived, not user input: lowercased, accent-stripped concatenation of `description` and `notes`, rebuilt whenever either changes; used for case- and accent-insensitive search |
| `removed_at` | timestamp, nullable | Set when the owner removes the transaction; cleared when restored |
| `created_at` / `updated_at` | timestamps | Standard |

### Derived Behavior

- **Counts toward balance**: `status = effective` and `removed_at is null`.
- **Future dated**: `transaction_date` is after the current calendar date.
- **Archived association retained**: the referenced account or category has a
  non-active status but the transaction keeps referencing it unchanged.

### Indexes

- `(user_id, transaction_date)` for the newest-first owner history and date
  range filters.
- `(user_id, status, removed_at)` for status filters and the removed view.
- `(user_id, search_text)` prefix support for text search scoped to the owner.
- `(financial_account_id)`, `(category_id)` for association lookups.

## TransactionType

| Value | Meaning | Balance effect when the transaction counts |
|-------|---------|--------------------------------------------|
| `income` | Money entering the account | Adds `amount_centavos` |
| `expense` | Money leaving the account | Subtracts `amount_centavos` |

The enum is closed for this feature and extensible by adding values with their
own rules. Stored rows keep their original meaning when a new value is added.

## TransactionStatus

| Value | Meaning | Balance effect |
|-------|---------|----------------|
| `pending` | Planned or not yet settled | None |
| `effective` | The financial movement has occurred | Applies the type's effect |

### State Transitions

| From | To | Allowed | Effect |
|------|----|---------|--------|
| `pending` | `effective` | Yes | Applies the movement once |
| `effective` | `pending` | Yes | Removes the movement once |
| any | same | Yes | No change |

Removal and restoration are a separate lifecycle:

| From | Action | To | Effect |
|------|--------|----|--------|
| active (`removed_at` null) | Remove | removed (`removed_at` set) | Removes the movement once if it was effective; the record is retained |
| removed | Restore | active, `status` chosen | Applies the movement once if the restored status is effective |
| removed | Edit any field | removed (unchanged) | Rejected with a state conflict; the transaction must be restored first |

Repeating remove on a removed transaction, or restore on an active transaction,
is a state conflict and changes nothing.

### Date-Edit Rules

- Recording a transaction dated in the future stores it as `pending`.
- Editing a `pending` transaction's date, in either direction, leaves it
  `pending`.
- Editing an `effective` transaction's date into the future keeps it
  `effective` and keeps its balance effect. The response states that the
  transaction remains effective and still affects the balance, and the user can
  choose to set it `pending` instead.
- No date edit changes `status` on its own.

## Validation Rules

- `description`: required, trimmed, 1–200 characters, at least one
  non-whitespace character.
- `notes`: optional, trimmed, at most 1000 characters, empty becomes `null`.
- `amount_centavos`: required integer, `>= 1`, `<= 99_999_999_999`. Rejects
  zero, negative, non-integer, over-precision, and out-of-range values.
- `transaction_date`: required, valid calendar date in
  `1900-01-01 .. 2100-12-31`.
- `type`: required, `income` or `expense` only.
- `status`: optional on create and update, `pending` or `effective` only.
- `financial_account_id`: required; owner's account; active for a new or
  changed association; an archived account is rejected for new associations but
  may be kept unchanged.
- `category_id`: required; category available to the owner; classification MUST
  match the type (`income` → income category, `expense` → expense category);
  active for a new or changed association; an archived category is rejected for
  new associations but may be kept unchanged.

## Relationships

- **User → Transactions**: one to many. Every transaction belongs to exactly one
  user through its financial context, and every read or write is scoped to that
  user.
- **FinancialAccount → Transactions**: one to many. The account stores
  `initial_balance_centavos` and the materialized
  `current_balance_centavos`, and its `has_financial_movements` flag becomes
  true on the first associated transaction.
- **Category → Transactions**: one to many. A category's `classification` must
  match the transaction type, and its `has_financial_transactions` flag becomes
  true on the first associated transaction.

## Balance Reconciliation Rules

For the account referenced by a transaction:

```text
effective_effect(transaction)
  = status is effective and removed_at is null
      ? (type is income ? +amount_centavos : -amount_centavos)
      : 0

new_current_balance
  = current_balance_centavos - effective_effect(before) + effective_effect(after)
```

Rules:

- Reconciliation runs inside one database transaction with the account row
  locked, so concurrent writes cannot interleave.
- Reconciliation runs after every action that can change a transaction's
  effect: create, update of any effect-relevant field, status change, removal,
  and restoration.
- Moving a transaction between accounts reconciles both accounts: the previous
  account loses the previous effect and the new account gains the new effect.
- Changing the type reconciles the same account twice: the previous type's
  effect is removed and the new type's effect is applied.
- A pending or removed transaction contributes zero, so a balance is always the
  account's initial balance plus the effect of its effective, non-removed
  transactions.
- Removing or archiving accounts and categories never changes balances, because
  account lifecycle does not alter the transactions' own effect.
- Archiving an account or category does not change any associated transaction's
  status or removal state; pending transactions stay pending and keep
  contributing zero, and they remain editable while keeping that archived
  association.

## Filtering and Search Shape

Supported filter inputs, combined with AND semantics:

| Input | Meaning |
|-------|---------|
| `from`, `to` | Inclusive transaction-date range |
| `type` | `income` or `expense` |
| `financial_account_id` | The owner's account, active or archived |
| `category_id` | A category available to the owner, active or archived |
| `status` | `pending` or `effective` |
| `q` | Free text matched against description and notes, case- and accent-insensitive |
| `view` | `active` (default) or `removed` |
| `page`, `per_page` | Server-side pagination; `per_page` defaults to 50 and is capped at 50 |

Ordering is newest first: `transaction_date` descending, then `id` descending so
same-day entries stay in creation order.

## Frontend-Facing Derived Values

- `amount_display`: Brazilian formatted value derived from `amount_centavos`
  (for example `R$ 1.234,56`).
- `type_display`: localized label plus a non-color sign or icon so income and
  expense are distinguishable without color.
- `is_future_dated`, `can_remove`, `can_restore`, `account_is_archived`,
  `category_is_archived`: derived flags used to explain state and to disable
  unavailable actions with a reason.
- `matching_count`: `meta.total` from the paginated history response, shown with
  the active filters.
