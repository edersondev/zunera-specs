# Quickstart: Financial Reports

## Read and branch gate

1. Confirm `016-financial-reports` in specs, `../zunera-backend`, and `../zunera-frontend` before editing an application.
2. Read [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [API contract](contracts/financial-reports-api.yaml), and `docs/design/{design-foundation,app-shell,navigation,components}.md`.
3. Finish and test backend contract, authorization, validation, money rules, and the paid-refund recognition repair before frontend work. No new package or report-specific financial table is planned.

## Backend path

1. Add tests proving a credit event on a fully/partly paid source purchase reduces that purchase's recognized installment spending in its original periods/categories while card credit and account cash effects remain correct. Repair the shared recognition projection used by Dashboard, Budgets, Financial History, and Reports. Cover centavo allocation across installments and idempotent replay.
2. Add protected Reports reads matching the contract. Resolve current, previous, selected completed historical-month, custom, and comparison periods in Sao Paulo business-date context. Reject `month` outside historical-month mode and custom dates outside custom mode. Validate filters, target metrics, cursor, owner-scoped account/category IDs, and incompatible type/category combinations. Return only owned data.
3. Build one signed recognized-contribution read layer from effective ordinary transactions, closed card installments, and traceable card credit-event allocations. Derive summary, evolution, category distributions, and both comparison periods from identical scope. Exclude recurrence templates, pending activity, transfers, settlements, and goal actions from income/expenses.
4. Build account rows with direct attributed income/expenses/net flow plus distinct effective transfer in/out and card statement settlement. Keep card installment expense unassigned to paying financial accounts.
5. Add cursor-paged contribution detail with all-record total. Verify total equals each summary/category/account metric even when more rows exist than one page. Preserve archived identity and card source links.
6. Measure 10,000-record/100-category/50-account fixtures, review query plans, and add only necessary source indexes. Complete the backend gate before frontend work.

Preferred backend checks from `../zunera-backend/` with live development stack:

```bash
docker exec zunera-backend-app-1 php artisan test --filter=FinancialReport
docker exec zunera-backend-app-1 php artisan test --filter=CreditCard
docker exec zunera-backend-app-1 php artisan test --filter=FinancialDashboard
docker exec zunera-backend-app-1 php artisan test --filter=Budget
docker exec zunera-backend-app-1 php artisan test
docker exec zunera-backend-app-1 vendor/bin/pint --format=agent
```

Run contract validation against `contracts/financial-reports-api.yaml`. Docker socket access may require sandbox escalation. Use MySQL in the development stack for refund/aggregation behavior that SQLite cannot prove.

## Frontend path

1. Add Reports route, navigation, localized labels, `reportsService`, one canonical URL-backed scope/store, and stale-request cancellation with the initial summary. Reuse that scope for later drill-down, filters, and period controls. Do not calculate authoritative money in the browser.
2. Add period selector, filter bar, summary, evolution, separate income/expense categories, account activity, comparison, and paged contribution drawer. Reuse existing formatters, chart wrapper, theme tokens, and Element Plus controls. Provide visible numeric alternatives to charts.
3. Show exact current/prior dates, filtered-scope labels, unavailable comparison percentages, archived identities, transfer/settlement separation, and all required empty/error states. Distinguish a section's unavailable/null response from an available empty list; keep reliable sibling sections usable. Keep report-specific card detail reachable rather than relying on Transactions' card row click.
4. Test service/state/components and isolated browser journeys. Verify PT-BR/English, keyboard/assistive-technology use, 320 px viewport, 200% zoom, and light/dark/system themes.

Frontend checks from `../zunera-frontend/`:

```bash
npm run test:unit -- --run
npm run lint
npm run build
CI=1 npm run test:e2e -- e2e/financial-reports.spec.js
```

`npm run lint` currently uses fix mode and may edit files; inspect changes afterward.

## Acceptance walkthrough

1. Set business date September 26, 2026. Select current month and year; verify September 1–26 and January 1–September 26 in every section. Select previous month/year and August 15–September 14 custom range; verify inclusive boundaries and prior comparison ranges. Navigate to completed July 2026 and verify July 1–31 compares with June 1–30; select July 1–31 as custom and verify its preceding equal-day comparison is May 31–June 30. Return to current month and verify month-to-date boundaries.
2. In September, add effective R$ 5.000,00 income and R$ 3.000,00 direct expense. Verify result R$ 2.000,00, interval/category sums, drill-down totals, and Dashboard parity. Test income-only, expense-only, empty month, and filtered-empty states.
3. Transfer R$ 300,00 from A to B and pay a card statement R$ 100,00 from A. Verify neither raises income/expenses. Account A shows transfer out R$ 300,00 and settlement R$ 100,00 separately; B shows transfer in R$ 300,00. Pending movements do not appear as realized.
4. Purchase by card August 28 on statement closing September 10. Before closing it is absent from Reports; after closing its installment enters September expense once. Paying statement later does not add expense. A later full or partial refund, including after source statement is paid, restates the source installment period/category and shows a negative traceable contribution without creating ordinary income.
5. Generate ordinary recurring pending occurrence and recurring card purchase. Verify rule/generation creates no second activity; only effective ordinary transaction or closed card installment counts. Goal allocation/release leaves report money unchanged. Correct, remove, and restore an ordinary transaction; verify report reflects current effect once.
6. Filter by account, category, and income/expense type separately and together. Verify all sections and detail use same filters, account filter does not assign card purchase to later paying account, and reset restores full scope. Try foreign account/category ID and confirm no disclosure.
7. Compare September with August; compare an in-progress March 1–31 against February 1–28 and verify exact dates, signed absolute differences, and unavailable percentages for unequal day counts. Verify prior zero/both zero/negative result/sign crossing. Category changes use neutral language.
8. Populate 10,000 records, 100 categories, and 50 accounts. Measure SC-005 latency, reach all detail pages, and sum every signed contribution to the reported total. Inspect dominant/tiny category readability and responsive/accessible alternatives.

## Outcome evidence

- Record centavo-level reconciliation and parity fixtures, including paid-source card refunds and archived historical identities.
- Record at least 20 representative warm navigations for the 10,000-record fixture and calculate the share meeting the 5-second target.
- For SC-002/SC-003, run uncoached tasks with at least 10 representative users; record period/result/largest-category identification within 60 seconds and source/refund tracing within 2 minutes. At least 90% must pass each task. Store anonymized results in a feature checklist during implementation; do not collect credentials or personal transaction contents.
