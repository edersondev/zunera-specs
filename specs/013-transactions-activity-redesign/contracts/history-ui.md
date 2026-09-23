# Existing Interface Contracts

- `GET /api/v1/financial-history` remains unchanged. Existing `from`, `to`, `q`, `type`, `status`, `financial_account_id`, `category_id`, `include`, `view`, `page`, `per_page` drive list; `meta.total` and pagination remain authoritative. `meta.totals` is lifetime-wide and must not label a month.
- `GET /api/v1/financial-dashboard/summary` remains unchanged. `preset=custom&from=YYYY-MM-DD&to=YYYY-MM-DD` gives realized period summary. It does not accept search, category, account, type, or status filters.
- Existing transaction and transfer detail/mutation endpoints remain unchanged. No new action or API field.
- UI contract: one expanded composite-key entry, native button header with `aria-expanded` and `aria-controls`, region content, existing action events, and no action-triggered header toggle.
