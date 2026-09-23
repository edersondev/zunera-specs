# Data Model: Presentation Only

- **History entry**: Existing `movement_kind`, `id`, `movement_date`, description, amount, status, and type-specific account/category/card/recurrence fields. No persistence change.
- **Period**: Existing inclusive `from`/`to` query bounds; UI mode derived as full month or custom range.
- **Expansion**: Route-local nullable composite key (`movement_kind:id`), never stored remotely.
- **Summary**: Existing Dashboard `realized_income`, `realized_expenses`, and `financial_result`; shown only for complete date bounds without non-date filters.
