# Data Model: Category Management

## Category

- `id`: Internal numeric identifier exposed only through authorized category
  resources.
- `origin`: `system` for a system default or `personal` for a user-created
  category. Immutable after creation.
- `user_id`: Required for `personal`; absent for `system`. A personal category
  belongs to exactly one user.
- `name`: Required user-visible category name. System default names are seeded;
  personal names are user-entered.
- `normalized_name`: Internal comparison value produced by trimming surrounding
  whitespace, collapsing repeated internal whitespace, and case- and
  accent-insensitive normalization.
- `classification`: `income` or `expense`. Required. Immutable for system
  defaults and immutable for a personal category once
  `has_financial_transactions` is true.
- `color`: Applied categorical palette identifier. Personal categories may use
  the permitted choices or receive the default; system defaults use their seeded
  visual identity. It never indicates financial polarity or status.
- `icon`: Applied permitted category icon identifier. Personal categories may
  use the permitted choices or receive the default; system defaults use their
  seeded visual identity.
- `status`: `active` or `archived`. System defaults are always active and
  read-only. Only personal categories may change status.
- `archived_at`: Timestamp set only when a personal category is archived.
- `has_financial_transactions`: False when no financial transaction has used
  the category; true once a transaction association exists. It protects
  classification from subsequent changes until the future transaction domain
  supplies the underlying association.
- `created_at` / `updated_at`: Audit timestamps.

## CategoryOrigin

- `system`: Global seeded default, non-user-owned, active, and read-only.
- `personal`: User-created category, visible only to its owner and subject to
  active/archived lifecycle behavior.

## CategoryClassification

- `income`: Category for income organization.
- `expense`: Category for expense organization.

No transfer, arbitrary, or user-defined classification is allowed in this
feature.

## CategoryStatus

- `active`: A system default or active personal category. Included in normal
  category selection.
- `archived`: Personal category excluded from normal new-transaction selection,
  retained for historical association, and eligible for restoration subject to
  conflict checks.

### State Transitions

```text
personal active --archive--> personal archived
personal archived --restore without active-name conflict--> personal active
personal active --archive when already archived--> rejected state conflict
personal archived --restore when already active--> rejected state conflict
system active --any mutation--> rejected read-only state conflict
```

Permanent deletion is not exposed.

## System Defaults

| Classification | Required default names |
| --- | --- |
| Expense | Housing; Food; Transportation; Health; Education; Entertainment; Shopping; Bills and utilities; Taxes; Other expenses |
| Income | Salary; Freelance or services; Investments; Gifts; Refunds; Other income |

All 16 defaults are seeded as `system` and active. They are returned in every
authenticated active-category list with their system origin. They are absent
from archived lists because they cannot be archived.

## Relationships

- A user owns many personal categories.
- A personal category belongs to exactly one user.
- System default categories belong to no individual user and are available to
  every authenticated user.
- Future financial transactions may belong to exactly one category. They keep
  that association when the category is archived.

## Validation and Integrity Rules

- `name` is required, non-whitespace, displayable, and limited to 120
  characters. Normalize it before conflict checks and persistence.
- `classification` is required and limited to `income` or `expense`.
- On create, personal category `origin`, owner, and active status are assigned
  by the system, not accepted from user input.
- On update, a personal category's classification is accepted only while
  `has_financial_transactions` is false; all other allowed fields retain normal
  validation.
- Color and icon are nullable on input but responses always expose the applied
  permitted default or selected value. The permitted color identifiers are
  `teal`, `blue`, `violet`, `amber`, `rose`, and `cyan`; these are categorical
  design choices only.
- A personal active category's tuple of owner, classification, normalized name,
  and active state is unique. Persistence enforcement protects concurrent
  personal create, rename, classification-change, and restore operations.
- The category service must additionally reject any personal active candidate
  whose classification and normalized name match a system default.
- Archive is valid only for an owned active personal category. Restore is valid
  only for an owned archived personal category whose proposed active state has
  no personal or system-default conflict.
- Unknown and other-user personal IDs are not found for privacy. A visible
  system category can be read but mutation returns a read-only state conflict.
