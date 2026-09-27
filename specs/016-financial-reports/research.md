# Research: Financial Reports

**Date**: 2026-09-26 | **Spec**: [spec.md](spec.md)

## Decision 1 — One recognized-contribution definition

**Decision**: Build a shared read projection of signed, dated contributions for ordinary effective transactions and recognized card installments. Use the same contribution set for summary, evolution, categories, comparisons, and drill-down. Reuse established Dashboard/ Budget recognition rules rather than Financial History aggregate totals.

**Rationale**: `DashboardSummaryService`, `DashboardExpenseDistributionService`, `DashboardEvolutionService`, and `RecurringCardExpenseProjection` already include effective ordinary transactions plus closed-statement installments. `FinancialHistoryService::list` includes card entries, but its `totals()` sums ordinary transactions only. Reusing those history totals would understate spending; independent per-widget queries would risk disagreement. Card expenses use statement closing date after statement finalization, not purchase or payment date. Transfers, goals, templates, expected entries, and card settlement remain outside realized income/expense.

**Alternatives considered**: Copy Dashboard calculations into Reports (drift risk); sum Financial History totals (omits card expense); persist a second reporting ledger (duplicates the financial model).

## Decision 2 — Repair paid-source refund recognition before Reports

**Decision**: In the backend phase, make recognized card spending reflect each source purchase's full accepted credit-event amount, allocated across its installments in original sequence under Spec 010, whether the affected statement is unpaid or already paid. Keep obligation settlement and card-credit application behavior separate. Move Dashboard, Budget, financial-history card presentation, and Reports onto that same authoritative recognized net projection; add cross-feature regression tests.

**Rationale**: `CreditCardCreditEventService::applyEvent` currently increments `credit_adjustment_centavos` only for unpaid source installments. A refund after full payment becomes card credit or pays other statements without reducing the source installment's recognized spending. The implemented Credit Cards specification requires the source installment periods and categories to be restated by credit events, including post-payment refunds. Reports cannot fix this locally without conflicting with Dashboard and Budgets. Preserve existing card-credit and cash-account effects while correcting spending recognition.

**Alternatives considered**: Count the refund as income (contradicts card rules); subtract card credit only inside Reports (breaks reconciliation); rewrite past payment/statement cash movements (unnecessary and incorrect). Design the shared allocation so accepted event totals cannot exceed uncredited purchase principal and every centavo is assigned once. Reassess whether a derived allocation suffices before adding persistence; no report-only table is planned.

## Decision 3 — Read-only report projections, not stored balances

**Decision**: No report-specific financial persistence or report-persistence migrations are planned. Source-table index migrations remain conditional on measured query plans. Define a report-scope value object and read-only contribution/query layer; use a repository for complex owner-scoped, cross-source aggregation and paginated traceability. Aggregate money as integer BRL centavos, including signed correction contributions.

**Rationale**: Report totals must immediately reflect edits, removal/restoration, archived identities, card credit events, and statement closing. A stored snapshot would need invalidation across many features and could silently become stale. Grouped queries and bounded pagination suit the 10,000-record acceptance scale. Laravel 13 documents query-builder grouping/aggregation and cursor pagination: [queries](https://laravel.com/docs/13.x/queries), [pagination](https://laravel.com/docs/13.x/pagination).

**Alternatives considered**: Materialized summary tables (extra synchronization); loading every record in the browser (unbounded data); one service per widget with unrelated filters (mismatched scope).

## Decision 4 — One report scope and explicit comparison calendar

**Decision**: Validate a single period/filter scope for every read. Derive current month/year against Zunera's America/Sao_Paulo business date. Support a selected completed historical month for navigation beyond the previous-month shortcut. Derive previous period on the backend: preceding full calendar unit for completed month/year and selected historical month; matching prior calendar dates for in-progress month/year with shorter-month/leap-day cap; equal-duration preceding range for custom dates, including a custom range that spans a full month. Return both exact inclusive ranges and day counts. Suppress percent change if day counts differ, previous value is nonpositive, or result crosses sign.

**Rationale**: All sections and drill-down must share one effective scope. Explicit server-derived comparison boundaries prevent browser locale drift and misleading percentages. The user confirmed the partial-calendar rule during Clarify.

**Alternatives considered**: Client-calculated comparison dates (timezone disagreement); always adjacent equal-day ranges (less intuitive month-to-date); full prior month against partial current month (misleading).

## Decision 5 — Account movement stays separate

**Decision**: Account-attributed income/direct expense and their net financial flow are distinct from effective transfer inflow/outflow and effective card-statement settlement paid from that account. Card installment spending remains workspace-wide and category-attributed, never assigned to the paying financial account. Account filter does not turn transfers/settlements into income or expense. Income, expense, and category filters suppress these uncategorized movements and their detail; removing those filters restores the separate figures.

**Rationale**: This follows existing account and card semantics and the Clarify answer requiring separate movement figures. A user can explain why an account moved without altering financial result. The response must explain why account direct-expense sums can differ from workspace-wide expense totals.

**Alternatives considered**: Treat settlement as categorized expense (double counts); allocate purchase to later paying account (retroactive and ambiguous); omit transfers/settlements (fails account-activity explanation).

## Decision 6 — A coherent overview plus paged contribution detail

**Decision**: Expose a protected, read-only report overview for one scope and a protected, paged contribution detail for a selected metric under the same scope. Overview carries summary, evolution, category distributions, account activity, and comparison with explicit period/filter metadata. Detail carries signed contributions, all-record total, and stable pagination. Use existing `auth:sanctum`, session lifetime protection, Form Requests, API Resources, and owner-scoped reads. The current product has user ownership rather than a separate workspace table; that owner boundary implements the spec's workspace isolation.

**Rationale**: One overview response gives all visible sections a common effective scope and avoids mixed-period presentation. Detail does not return thousands of rows with overview. Foreign account/category IDs are rejected without revealing their identity; archived historical labels remain readable.

**Alternatives considered**: Several independent section requests without a shared scope token (mixed-period risk); export all source rows in overview (payload and responsiveness risk); client-side totals (trust and precision risk).

## Decision 7 — Reuse existing Vue and design system

**Decision**: Add a protected Reports route and navigation item. Keep URL query as canonical period/filter context; a focused Pinia store holds fetched overview/detail and request generation, while services alone call the API. Use JavaScript `<script setup>` per current project convention, Element Plus for controls, Tailwind/semantic tokens, existing BRL/date/percentage formatters, and existing ApexCharts wrappers only where they aid reading. Every visualization gets a visible numeric/text alternative. Use a report-specific detail drawer/list for ordinary and card contributions because Transactions currently does not open `credit_card_expense` rows.

**Rationale**: Existing Dashboard and Transactions patterns cover formatting, accessible controls, chart themes, month navigation, and URL filters. Vue 3.5 watcher cleanup can cancel stale async fetches when period/filter context changes: [Vue watcher guidance](https://vuejs.org/guide/essentials/watchers). No new package is needed. Follow `docs/design/design-foundation.md`, `app-shell.md`, `navigation.md`, and `components.md`.

**Alternatives considered**: New chart package (unneeded); drill-down solely through existing Financial History (card row cannot open and totals exclude cards); component-local fetches for every section (mixed scope and duplication).

## Decision 8 — Backend verification gate before frontend

**Decision**: Verify authorization, validated scope, centavo reconciliation, card paid/unpaid refund restatement, period boundaries, all filters, paged detail, Dashboard/Budget parity, and 10,000-record read behavior before frontend implementation. Then verify frontend service/store/component behavior, accessible charts and controls, route state, and isolated browser journeys.

**Rationale**: Financial integrity is the primary risk. Contract-first backend completion follows the constitution and lets frontend consume stable shapes. No uploads or new secrets are involved.

**Alternatives considered**: UI-first delivery (hard to verify semantics); only unit tests (misses cross-feature and UI-to-API behavior).
