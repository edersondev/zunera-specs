# Data Model: Credit Cards

## Ownership and money

Every persisted card-domain record is owned through its `CreditCard.user_id`.
All money is integer centavos with `currency_code: BRL`. Foreign records are not
discoverable. Derived balances, statuses, and reporting totals are never stored
as independent financial truth.

### CreditCard

| Field | Rules |
|---|---|
| `id`, `user_id` | Stable identity and required owner |
| `name`, `institution_name` | Required trimmed name; required institution |
| `last_four`, `color`, `icon` | Nullable non-sensitive display information only |
| `credit_limit_centavos` | Integer 1–99,999,999,999 |
| `closing_day`, `due_day` | Integer 1–31; actual dates clamp to month end |
| `status`, `archived_at` | `active` or `archived`; archive requires zero outstanding and zero card credit |
| timestamps | Audit only |

Derived card summary: `used_credit` is net unpaid principal, never negative;
`card_credit` is unapplied effective credit-event value; `available_credit` is
limit minus used credit plus card credit and may exceed stated limit.

### CreditCardPurchase and CreditCardInstallment

| Field | Rules |
|---|---|
| Purchase owner/card/category | Required owned active card and eligible expense category at creation |
| `description`, `notes`, `purchase_date` | Required nonblank ≤200 description; nullable note ≤1,000; 1900–2100 date |
| `total_amount_centavos`, `installment_count` | Positive total; count 1–360; interest-free only |
| Installment `sequence`, `amount_centavos` | Unique purchase/sequence; positive; sum exactly equals total; early rows receive remainder cents |
| Installment `statement_id` | Required cycle allocation; sequence 1 uses purchase cycle, later sequence later cycles |
| recognition state/date | Derived pending/effective and statement closing date; no payment-date recognition |
| category/card snapshots | Preserve historical name/status/display association after archive |

Purchase direct edit is available only if all linked statements remain Open.
Changes reallocate future/open installments atomically. It never creates a
financial-account movement.

### CreditCardStatement

| Field | Rules |
|---|---|
| `credit_card_id`, `closing_date` | Unique pair; owner follows card |
| `period_from`, `period_to` | Inclusive cycle boundaries; closing date is period end |
| `due_date` | First configured due-day occurrence strictly after actual closing |
| `original_amount_centavos` | Sum allocated installment gross amounts |
| `credit_adjustment_centavos` | Sum applied source credit events |
| `paid_centavos`, `card_credit_applied_centavos` | Effective settlements only |
| `net_amount`, `outstanding_centavos` | Derived nonnegative values; `outstanding = max(0, net − paid − card-credit-applied)` |
| status | Derived `open`, `closed`, `partially_paid`, `paid`, `overdue` |

Active card's current cycle returns a synthesized Open zero statement until it
has activity. Archived cards receive no new current statement. Existing
activity cycles materialize a unique historical statement. Once allocated, a
statement's period, closing date, due date, and installment assignment are
immutable; later card billing-configuration changes affect only later purchases.

### CreditCardStatementPayment

| Field | Rules |
|---|---|
| statement/card/user/account | Required same-owner statement and active account for new/change |
| `amount_centavos`, `payment_date`, `notes` | Positive amount no more than current outstanding; date 1900–2100; nullable note |
| `status`, `removed_at` | Existing `pending`/`effective` plus recoverable removal lifecycle |
| account balance effect | Effective/non-removed debit only; pending/removed no debit; effective settlement may overdraw account like ordinary expense |
| idempotency reference | Mutation owner/key/fingerprint/replay source |

Historical archived account relation remains readable but cannot be selected for
new/reassigned payment. Payment is settlement, has no category, and cannot join
expense totals.

### CreditCardCreditEvent and CreditCardCreditApplication

| Field | Rules |
|---|---|
| event source | Required owned purchase; reason `cancellation`, `refund`, or `correction` |
| `amount_centavos`, `event_date`, `notes` | Positive no more than source uncredited amount; date 1900–2100; nullable notes |
| purchase installment applications | Ordered original installment sequence; exact centavo allocation; immutable audit rows |
| statement applications | Reduce source statement first; remaining card credit applies oldest unpaid by due/closing order |
| residual card credit | Visible, reusable, no direct financial-account debit/credit |

Credit events preserve original purchase and statement rows. Recalculation after
payment/credit event edit, removal, or restoration writes no duplicate money
effect and never makes statement balance negative.

## Relationships and authoritative projections

```text
User 1 ── * CreditCard 1 ── * Purchase 1 ── * Installment * ── 1 Statement
                                  │                 │
                                  └── * CreditEvent ─┴── * CreditApplication
Statement 1 ── * StatementPayment * ── 1 FinancialAccount
Purchase * ── 1 Category
```

| Consumer | Card-source rule |
|---|---|
| Financial account balance | Effective non-removed statement payment debits its linked account once |
| Budgets | Net effective installment in statement closing month; pending open/current-future installment is expected only |
| Dashboard summary/distribution/evolution | Same net effective installment source; payment excluded |
| Dashboard card projection | Separate outstanding/card-credit/available-credit/upcoming-statement read; never part of cash balance |
| Financial History | Discriminated `credit_card_expense` installment entry; card settlement payment excluded |
| Card detail | Gross installment, credit adjustments, payments, card-credit applications, and status remain traceable |

## State transitions

```text
Statement: Open ──closing──> Closed ──partial settlement──> Partially paid
    │                         │                                  │
    └──full settlement────────┴──────────────────────────────────┴──> Paid
Closed/Partially paid ──past due with outstanding──> Overdue

Payment: Pending <──future date/new choice──> Effective ──remove──> Removed
                                           └────restore───────┘
```

All transitions validate current state after deterministic locks. Paid remains
Paid after due date. Credit application can take Closed/Partially paid/Overdue to
Partially paid or Paid but cannot reopen an Open statement or create negative
outstanding.
