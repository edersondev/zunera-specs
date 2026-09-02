# Data Model: Financial Accounts

## FinancialAccount

- `id`: internal numeric primary identifier exposed in the API; every exposed
  value is scoped to the signed-in user.
- `user_id`: owner identifier. Required; every query and action is scoped to this
  owner.
- `name`: user-visible account label chosen by the user, separate from financial
  institution text. Required, trimmed, and unique among active accounts owned by
  the same user after normalization.
- `normalized_name`: comparison value derived from `name` for active-name
  uniqueness; trims surrounding spaces, collapses repeated internal spaces, and
  compares without case or accent differences. Not exposed to users.
- `account_type`: one of `checking`, `savings`, `cash_wallet`, `investment`,
  `digital`, or `other`.
- `institution_name`: optional user-entered text describing the financial
  institution; separate from `name` and no catalog match is required.
- `color`: optional predefined accessible color token; defaults when omitted.
- `icon`: optional predefined account icon identifier; defaults when omitted.
- `initial_balance_centavos`: signed integer amount in BRL centavos. Required on
  creation, limited to +/-999,999,999,999 centavos, and editable only while no
  financial movements exist.
- `current_balance_centavos`: signed integer amount in BRL centavos. Equals
  `initial_balance_centavos` until future movement features update balances and
  stays within the supported per-account range.
- `currency_code`: fixed value `BRL` for this feature.
- `status`: `active` or `archived`.
- `archived_at`: timestamp when account entered archived state; null while
  active.
- `has_financial_movements`: boolean state indicating whether future movement
  records exist for the account. False for accounts created by this feature
  before movement features exist.
- `created_at` / `updated_at`: audit timestamps.

## AccountType

- `checking`: checking account.
- `savings`: savings account.
- `cash_wallet`: physical cash or wallet.
- `investment`: investment account.
- `digital`: digital account.
- `other`: account type not represented by the predefined financial categories.

Account types classify financial nature only. They do not define transaction,
transfer, investment, or balance-calculation behavior.

## AccountStatus

- `active`: visible in normal account lists, eligible for new financial-operation
  choices, included in active combined balance, and editable subject to normal
  field rules.
- `archived`: hidden from normal new-operation choices and excluded from active
  combined balance, but still readable in archived/history contexts and
  restorable when doing so does not violate active-name uniqueness.

### State Transitions

```text
active --archive--> archived
active --archive when this is the last active account--> archived + zero active summary
archived --restore without active-name conflict--> active
active --archive when already archived--> rejected state conflict
archived --restore when active-name conflict exists--> rejected state conflict
```

Permanent deletion is not exposed by this feature.

## BalanceSummary

- `active_account_count`: number of active accounts owned by the signed-in user.
- `active_combined_balance_centavos`: sum of `current_balance_centavos` across
  active accounts only; zero when the user has no active accounts.
- `currency_code`: fixed value `BRL`.

## Relationships

- A user owns many financial accounts.
- A financial account belongs to exactly one user.
- Future financial movement records may belong to one financial account and drive
  `has_financial_movements` and `current_balance_centavos`.

## Validation Rules

- Name is required, trimmed, user-visible, separate from financial institution,
  and must remain unique among the owner's active accounts after trimming
  surrounding spaces, collapsing repeated internal spaces, and ignoring case and
  accent differences. Database enforcement uses a partial unique index on
  `(user_id, normalized_name)` for active accounts so concurrent
  create/restore/rename operations cannot bypass service checks.
- Account type must be one of the supported account types.
- Institution name is optional user-entered text and must stay within a practical
  display length; empty institution text is rejected, and null or omitted means
  no institution is stored.
- Color and icon must be omitted or selected from predefined accessible choices;
  null or omitted means the default accessible choice is stored, and responses
  always include the applied color and icon.
- Initial balance is required on creation, accepted as BRL, converted to signed
  integer centavos, limited to +/-999,999,999,999 centavos, and must preserve
  exact centavo value.
- Initial balance updates are accepted only when `has_financial_movements` is
  false.
- Archive is accepted only for active accounts owned by the user.
- Archiving the user's last active account is allowed and leaves the active
  account summary at count zero and balance zero.
- Restore is accepted only for archived accounts owned by the user and only when
  the restored name would not duplicate another active account name owned by the
  same user.
