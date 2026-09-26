# Feature Specification: Financial Reports

**Feature Branch**: `016-financial-reports`  
**Backend Branch**: `016-financial-reports` (`../zunera-backend`)  
**Frontend Branch**: `016-financial-reports` (`../zunera-frontend`)  
**Created**: 2026-09-26  
**Status**: Ready for implementation  
**Input**: "Spec 011 — Financial Reports". Directory 011 already belongs to Transactions Month Navigator, so sequential Spec Kit directory 016 is used.

## Clarifications

### Session 2026-09-26

- Q: For an in-progress month or year, which preceding dates should Reports compare? → A: Match calendar dates in the preceding month or year, cap at the preceding period's last day, label exact ranges, and omit percentage changes when durations differ.
- Q: Should first-release account reports show transfers and card statement payments as separate movement figures? → A: Yes; show effective incoming/outgoing transfers and card statement settlements separately for each account, without treating them as income or expenses.
- Q: Should first-release Reports show expected activity? → A: No; show realized activity only, while excluding expected transactions and Open-statement installments until they become recognized under existing rules.
- Q: How should Reports handle a card refund after its source statement was fully paid, given current code differs from the Credit Cards specification? → A: Restore shared recognition to the established Credit Cards rule within this feature: reduce the source installment's recognized expense in its original period, while preserving separate card-credit and cash-settlement effects across Reports, Dashboard, and Budgets.
- Q: Should separate account transfer and card-settlement figures remain visible under income, expense, or category filters? → A: No; those uncategorized, non-income/expense movements appear only when neither transaction-type nor category filter is active, and still remain separate from realized income and expenses.
- Q: How should month navigation beyond the previous month compare periods? → A: A navigated completed historical month covers that full calendar month and compares with the full immediately preceding calendar month. Returning to the current month uses month-to-date rules. Custom ranges retain their equal-day preceding-range comparison even when their dates happen to span a full month.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Understand a Period's Result (Priority: P1)

As an authorized user, I want realized income, expenses, and result for one clearly labelled period so I can understand whether my finances gained or lost money.

**Why this priority**: Every analysis depends on accurate totals and dates.

**Independent Test**: Select a month with effective ordinary income/expenses and recognized card installments; reconcile all three figures with their authoritative source effects and equivalent Dashboard metrics under the same scope, then change periods.

**Acceptance Scenarios**:

1. **Given** September has R$ 5.000,00 realized income and R$ 3.000,00 realized expenses, **When** its report opens, **Then** result is positive R$ 2.000,00.
2. **Given** a period has R$ 500,00 income and R$ 800,00 expenses, **When** its report opens, **Then** result is negative R$ 300,00 and its meaning is readable without color alone.
3. **Given** a period contains only income, only expenses, or neither, **When** its report opens, **Then** the absent side is zero, result remains income minus expenses, and relevant empty guidance appears.
4. **Given** an effective own-account transfer, pending transaction, goal allocation, and goal release, **When** their dates fall in the period, **Then** none changes realized income, expenses, or result.
5. **Given** Reports and Dashboard use equivalent workspace, period, and financial scope, **When** both show a common realized metric, **Then** their amounts agree exactly.

---

### User Story 2 - Explore Evolution and Categories (Priority: P1)

As an authorized user, I want income and expense trends and separate category distributions so I can identify the drivers behind a period's result.

**Why this priority**: Trends and categories turn totals into financial understanding.

**Independent Test**: View dated, categorized ordinary and card activity; reconcile interval and category sums with the summary.

**Acceptance Scenarios**:

1. **Given** activity on several dates, **When** a short or long period is selected, **Then** labelled intervals distinguish realized income and expenses, and their sums match the summary.
2. **Given** Food contributes R$ 1.250,00 of R$ 4.000,00 expenses, **When** expense categories appear, **Then** Food shows R$ 1.250,00, its correctly formatted 31,25% share, and its position among the largest contributors.
3. **Given** realized income in multiple income categories, **When** income analysis opens, **Then** income categories appear separately from expenses and reconcile with income.
4. **Given** a purchase on August 28 has an installment on a statement closing September 10, **When** August and September are viewed after closing, **Then** that installment is realized in September, not August; purchase and payment dates do not move recognition.
5. **Given** a recognized installment and a later effective statement payment, **When** both periods are viewed, **Then** the installment contributes one expense in its recognition period and the payment adds none.
6. **Given** a card refund or post-closing correction, **When** the affected installment periods are viewed, **Then** category and evolution values reflect its established net adjustment once.
7. **Given** a recurring-income occurrence generates a pending transaction, **When** Reports opens, **Then** neither the rule nor pending occurrence counts as realized; after the transaction becomes effective, exactly one income contribution appears on its authoritative date.
8. **Given** a recurring-card occurrence generates a purchase on an Open statement, **When** Reports opens before and after statement closing, **Then** the rule and generation event never count as expenses; the Open-statement installment is absent from Reports and becomes realized exactly once in its closing period.
9. **Given** a R$ 1.000,00 recognized card installment is later partially refunded by R$ 200,00, **When** the affected period and category are viewed, **Then** recognized expense is R$ 800,00 and a later statement payment adds R$ 0,00 expense.
10. **Given** a R$ 1.000,00 card installment was recognized and its statement fully paid before a R$ 200,00 refund, **When** the affected period and category are viewed, **Then** recognized expense is R$ 800,00, the refund remains traceable, and any resulting card credit or application does not create income or another expense.

---

### User Story 3 - Trace a Reported Amount (Priority: P1)

As an authorized user, I want the source records behind category and account figures so I can investigate and trust each amount.

**Why this priority**: Financial totals need an explanation users can inspect.

**Independent Test**: Open a category or account amount and add all signed recognized contributions to reproduce its total.

**Acceptance Scenarios**:

1. **Given** Food reports R$ 1.250,00, **When** Food is selected, **Then** source records show recognized dates and signed contributions under the same period and filters, summing to R$ 1.250,00.
2. **Given** a card refund restates an earlier installment, **When** the category is opened, **Then** the adjustment and source purchase are identifiable, and the refund is not ordinary income.
3. **Given** an ordinary transaction is corrected, removed, or restored, **When** reports refresh, **Then** summary and detail show its current authoritative effect once, without stale copies.
4. **Given** hundreds of contributing records, **When** detail opens, **Then** all records remain reachable and the full set reconciles with the displayed total.

---

### User Story 4 - Compare Equivalent Periods (Priority: P2)

As an authorized user, I want the selected period compared with an explicit prior period so I can see changes in income, expenses, result, and categories.

**Why this priority**: Comparison supplies context for a single period's values.

**Independent Test**: Test completed, current, and custom periods; verify date labels, signed differences, and zero/negative percentage handling.

**Acceptance Scenarios**:

1. **Given** completed September is selected, **When** comparison appears, **Then** full September and August boundaries and realized income, expenses, and result are shown.
2. **Given** Food spent R$ 1.250,00 now and R$ 980,00 previously, **When** category comparison appears, **Then** absolute change is +R$ 270,00 with neutral wording.
3. **Given** previous income is zero and current income is positive, **When** change appears, **Then** absolute change is shown and percentage is unavailable, without infinity.
4. **Given** both values are zero, **When** change appears, **Then** absolute change is zero and percentage is unavailable.
5. **Given** previous result is negative or changes sign, **When** change appears, **Then** signed absolute change is correct and misleading percentage is omitted.
6. **Given** no prior-period activity, **When** comparison appears, **Then** both periods remain identified, previous totals are zero, and an explanatory no-previous-activity message appears.
7. **Given** month navigation selects completed March and February has only 28 days, **When** comparison appears, **Then** it shows March 1–31 against February 1–28, labels both exact ranges, shows absolute differences, and omits percentage changes because durations differ.
8. **Given** month navigation selects completed July 2026, **When** comparison appears, **Then** it shows July 1–31 against June 1–30, not an equal-day range beginning May 31; a separately selected July 1–31 custom range still compares with the preceding 31 inclusive days.

---

### User Story 5 - Analyze Accounts and Filter Scope (Priority: P2)

As an authorized user, I want account activity and filters so I can focus on relevant flows without mistaking filtered figures for my full finances.

**Why this priority**: Transfers, card settlements, and account filters need explicit meaning.

**Independent Test**: Apply account, category, and type filters separately and together; inspect every section, detail, scope label, and reset.

**Acceptance Scenarios**:

1. **Given** Account A receives R$ 1.000,00 income, pays R$ 200,00 direct expenses, sends R$ 300,00 to Account B, and pays a R$ 100,00 card statement, **When** its activity is viewed, **Then** attributed income and expenses are R$ 1.000,00 and R$ 200,00, net financial flow is R$ 800,00, and R$ 300,00 outgoing transfer plus R$ 100,00 card settlement are separate labelled movements.
2. **Given** Account A is selected, **When** reports update, **Then** summary, evolution, categories, comparison, and detail use the same account scope and visibly say they are filtered.
3. **Given** Account A later pays a card statement, **When** Account A is selected, **Then** the associated card purchase is not attributed to A's expense merely because A paid it.
4. **Given** Food is selected, **When** reports update, **Then** only eligible Food expenses contribute; unrelated income and expenses show zero where relevant.
5. **Given** income type is selected, **When** reports update, **Then** expense sections say no matching expenses and reset restores full period scope.
6. **Given** another workspace's account, category, or activity, **When** the user opens or filters Reports, **Then** no foreign identity, amount, or detail is disclosed.
7. **Given** Account A has effective transfers and card settlements, **When** the user applies an income, expense, or category filter, **Then** those separate movement figures and their detail are absent from the filtered report; removing type/category filters restores them without changing income or expense classification.

---

### User Story 6 - Navigate Periods and Empty States (Priority: P2)

As an authorized user, I want reliable period selection and explanatory empty states so I know exactly what the report includes.

**Why this priority**: Unclear scope and empty visualizations lead to false conclusions.

**Independent Test**: Choose all period presets and a custom range; inspect boundaries, report sections, filtered-out data, and high-volume data.

**Acceptance Scenarios**:

1. **Given** business date September 26, 2026, **When** current month or year is selected, **Then** report covers September 1–26 or January 1–September 26 and labels both boundaries.
2. **Given** custom range August 15–September 14, **When** it is selected, **Then** both dates are included in every relevant section and drill-down.
3. **Given** no realized activity, **When** report opens, **Then** exact zero summary values and an explanation appear instead of misleading distributions.
4. **Given** unfiltered activity exists but selected filters match nothing, **When** report updates, **Then** it states no records match filters and offers reset.
5. **Given** 100 categories, 50 accounts, and many transactions, **When** report opens, **Then** largest contributors remain findable, very small and large values remain legible, and all detail records remain accessible.
6. **Given** the user navigates from the current month to a completed historical month and back, **When** reports update, **Then** the historical month uses full calendar boundaries, the current month ends on the current business date, and every section and drill-down shows the selected period.

### Edge Cases

- An effective ordinary transaction redated into the future remains effective under Spec 004; Reports use its authoritative state and transaction date, never silently convert it to expected.
- An Open-statement installment stays expected in the source financial model even though its purchase exists. It is absent from first-release Reports; closing changes its existing installment effect to realized on closing date without adding another expense.
- Archived accounts, categories, and cards remain identifiable in valid historical activity.
- Empty categories and accounts need not dominate analysis. A zero denominator makes a share unavailable.
- A later refund or correction may restate an earlier period; subsequent views and comparisons use current authoritative history.
- A current-month or current-year comparison uses partial periods with both exact ranges labelled.
- One dominant category must not make smaller contributors unfindable. Negative and zero results must remain legible.
- A section error identifies unavailable information and permits retry while other reliable sections remain usable.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Reports MUST have a dedicated area inside the authorized financial workspace. Every report, filter choice, comparison, and detail MUST obey the same workspace isolation rules as existing financial features.
- **FR-002**: Reports MUST use established rules from Accounts, Categories, Transactions, Transfers, Recurring Transactions, Dashboard, Budgets, Credit Cards, Goals, and Recurring Credit Card Purchases. Reports MUST NOT establish independent balance, lifecycle, or recognition rules.
- **FR-003**: Reports MUST offer current month, previous month, current year, previous year, and inclusive custom date range; clearly show effective boundaries; and permit convenient navigation to older completed months. Current month/year end at current business date; completed previous month/year and navigated historical months cover full calendar periods. Returning to the current month restores month-to-date boundaries.
- **FR-004**: A period or filter change MUST update summary, evolution, categories, accounts, comparison, and detail consistently. No section MAY silently retain another scope.
- **FR-005**: Summary MUST show realized income, realized expenses, and result. Result MUST equal income minus expenses under the same scope, with positive, negative, and neutral meaning expressed without color alone.
- **FR-006**: Realized totals MUST include only financially effective ordinary income/expense transactions dated in the period and recognized card installment net effects under existing rules. Pending transactions, expected installments, recurrence definitions, and ungenerated occurrences MUST NOT contribute.
- **FR-007**: Own-account transfers MUST be excluded from income, expenses, and result at all scopes. In account analysis without a transaction-type or category filter, effective transfers MUST appear as separately labelled incoming/outgoing movement for involved accounts; pending transfers MUST NOT appear as realized movement.
- **FR-008**: Goal allocations, withdrawals, releases, and other goal-only actions MUST NOT change reported income, expenses, result, account balance, or account movement. An independent real financial transaction keeps its ordinary effect.
- **FR-009**: Ordinary recurring occurrences MUST affect Reports only through generated transactions and those transactions' authoritative state/date. Recurring card occurrences MUST affect Reports only through generated purchases' recognized installments. Definitions and generation events are not extra activity.
- **FR-010**: Each card installment MUST be attributed to calendar month containing its associated statement closing date, become realized on that closing date only after statement closes, and retain that date for reporting. Open-statement installments are expected. Purchase and payment dates MUST NOT replace this recognition date.
- **FR-011**: Statement payments and card-credit applications MUST NOT create additional income or expense. Refunds, cancellations, and post-closing corrections MUST reduce the source purchase's recognized installment expenses in their original periods/categories by the established signed net effect exactly once, including when source statements were already fully paid. They MUST remain traceable to source purchase and MUST NOT be reinterpreted as ordinary income. Any resulting card credit or settlement retains its separate existing financial effect; Reports, Dashboard, and Budgets MUST agree on recognized spending.
- **FR-012**: Effective ordinary transaction edits, removal, restoration, reclassification, and date changes MUST affect current reports according to existing lifecycle rules. Neither former nor corrected effect may be counted twice.
- **FR-013**: Evolution MUST distinguish realized income and expenses in clearly labelled, gap-free, non-overlapping intervals and MAY show result. Interval sums MUST equal summary. Use daily intervals through 31 days, weekly intervals for 32–93 days, and monthly intervals for longer periods, matching Dashboard; identify partial boundary intervals.
- **FR-014**: Expense categories MUST show identity, realized net amount, and share of all realized expenses in same scope, ordered so largest contributors are easy to identify. Zero denominators MUST show unavailable share. Zero-activity categories need not occupy primary view.
- **FR-015**: Income categories MUST appear separately, using existing income classification, with amounts and shares of realized income. Income and expense distributions MUST NOT be merged.
- **FR-016**: Archived category identities MUST remain readable in history. Card expense MUST use its authoritative purchase/installment category, not payment account or statement as a substitute.
- **FR-017**: Account analysis MUST show realized attributed income, direct expenses, and resulting net financial flow (income minus direct expenses) for each relevant financial account. When no transaction-type or category filter is active, it MUST separately show effective incoming/outgoing own-account transfers and effective card statement settlement paid from that account. These separate movements MUST NOT alter income, expenses, or result; they explain account activity without being presented as account balance. Pending transfers and payments MUST NOT appear as realized movement.
- **FR-018**: Card installments without financial-account attribution at recognition MUST remain in workspace-wide expense/category totals but MUST NOT be assigned to an account merely because that account later pays a statement. Reports MUST explain why account direct-expense sums can differ from workspace-wide expenses.
- **FR-019**: First-release Reports MUST show realized financial activity only. Expected ordinary transactions, Open-statement installments, pending transfers/payments, and ungenerated recurring occurrences MUST be absent from report values and drill-down. A future expected or forecast view, if specified separately, MUST remain distinct from realized values and MUST NOT call expected amounts spent.
- **FR-020**: Reports MUST show an explicit preceding comparison range. Completed month/year presets and navigated historical months compare with the complete preceding calendar month/year. Current month compares month-to-date with the same numbered dates in previous month, capped at its last day. Current year compares year-to-date with preceding year's same calendar dates, mapping leap-day end to February 28 when necessary. Custom range compares with the immediately preceding range of equal inclusive day count, even if its dates cover a full calendar month. Both exact ranges and their inclusive durations MUST appear.
- **FR-021**: Comparison MUST show current/prior realized income, expenses, and result plus signed absolute differences. Category comparison MUST show current/prior category spending and absolute difference for categories active in either period, with neutral wording. Percentage difference MAY appear only when periods have equal inclusive day counts, previous value is positive, and values have compatible nonnegative meaning. Unequal durations, previous zero/negative, or result sign crossing MUST show percentage unavailable. Both zero MUST show zero absolute difference and unavailable percentage.
- **FR-022**: Period, financial-account, category, and applicable transaction-type filters MUST be combinable, visibly active, and resettable. All relevant sections and detail MUST share them. Filtered totals MUST be labelled as filtered.
- **FR-023**: Account filter MUST include only income/direct expense authoritatively attributed to that account. When no type or category filter is active, it MUST show that account's effective transfers and card settlements separately as movement, not income/expense. A card installment MUST NOT enter account-filtered expenses solely through paying account.
- **FR-024**: Category filter MUST use existing record classification in selected and comparison periods. Incompatible category/type combinations yield zero and matching explanation. When an income, expense, or category filter is active, uncategorized transfer and card-settlement figures and their drill-down MUST be absent from the filtered report; clearing type/category filters restores those separate figures. Filters MUST NOT reclassify transfers or settlements.
- **FR-025**: Category and account amounts MUST offer source records with selected period, filters, recognized dates, and signed contributions preserved. Sum of all contributions MUST equal displayed total exactly, including adjustments. All contributing records MUST be reachable without creating a reporting copy of financial data.
- **FR-026**: Reports MUST distinguish no activity, no income, no expenses, no records matching filters, and no prior-period activity. Empty distributions MUST use explanatory states instead of misleading zero charts.
- **FR-027**: Reports MUST remain understandable with many categories/accounts/transactions, a dominant category, small/large BRL amounts, and negative/zero results. Largest contributors and all detail records MUST remain findable.
- **FR-028**: Reports MUST use Zunera's existing currency, decimal, date, month-name, and percentage conventions; visual differences MUST remain understandable without color alone.
- **FR-029**: Reports and Dashboard common realized metrics MUST reconcile exactly when workspace, period, filters, and recognition scope match. Any intentional difference in scope or included source MUST be labelled.
- **FR-030**: Initial release excludes custom report builders, formulas, predictive AI, investment analytics, tax/regulatory statements, public links, scheduled delivery, goal analytics, advanced budget-versus-actual analysis, advice, and bank-data enrichment.

### Security and Quality Requirements *(mandatory)*

- **SQR-001**: Every report read, filter, comparison, and detail MUST enforce existing workspace authorization. Invalid ranges, inaccessible accounts/categories, and unsupported filters MUST receive clear feedback without foreign data disclosure.
- **SQR-002**: Automated coverage MUST verify financial integrity, authorization, date boundaries, refunds/corrections, comparison zeros, filters, and detail reconciliation. Report-facing contract changes require contract coverage; critical summary-to-detail, filter, and comparison journeys require end-to-end coverage.
- **SQR-003**: Reports add no secrets or file uploads. Existing financial-data handling and retention apply; records remain private to authorized workspace users.

### Delivery Scope *(mandatory)*

1. **Backend** (`../zunera-backend`): Define report-facing contract, authorization, range/filter validation, authoritative financial semantics, traceable detail, and automated tests before frontend implementation. Internal design belongs to Plan.
2. **Frontend** (`../zunera-frontend`): Build Reports against verified backend contract: periods, summary, trends, categories, accounts, comparison, filters, drill-down, localization, accessibility, and tests. Presentation and state design belong to Plan.

### Key Entities *(include if feature involves data)*

- **Reporting period**: Selected inclusive business-date range and equivalent preceding range.
- **Report scope**: Authorized workspace plus active account, category, and type filters shared by every section.
- **Recognized financial contribution**: Existing financially effective income or expense effect with established date, category, account attribution where applicable, and signed amount.
- **Financial summary**: Realized income, expenses, and their difference under one period and scope.
- **Category contribution**: Category's recognized total and share of matching income or expense total.
- **Account activity**: Account-attributed income/direct expense and their net financial flow, plus separately identified effective incoming/outgoing transfers and card statement settlements.
- **Comparison**: Current/prior period amounts, signed absolute change, and percentage availability.
- **Source record**: Existing transaction, installment, transfer, payment, or card credit event explaining a contribution; no replacement financial record is created.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In every acceptance fixture, income, expenses, result, interval sums, category sums, and drill-down contributions reconcile to the cent with authoritative effects; transfers, statement payments, goal actions, and recurrence templates contribute zero to income/expenses.
- **SC-002**: At least 90% of usability-check participants identify selected period, realized result, and largest expense category without help within 60 seconds.
- **SC-003**: At least 90% of participants trace a chosen category amount to source records and identify an included refund/correction within 2 minutes.
- **SC-004**: Every period choice updates all period-sensitive sections with matching boundaries; comparison labels identify both exact periods in 100% of tested cases.
- **SC-005**: With 10,000 financial records, 100 categories, and 50 accounts in authorized scope, summary opens and any displayed total's contributing records are reachable within 5 seconds in at least 95% of attempts under normal test conditions.
- **SC-006**: In all zero-denominator and negative-result comparison tests, absolute differences remain accurate and no misleading percentage appears.

## Assumptions

- Zunera's current financial workspace boundary remains authoritative; Reports introduce no sharing.
- Current month/year end on current business date, consistent with Dashboard. Completed presets cover full calendar periods.
- Account filters cover financial accounts, not credit cards. Paying a statement does not assign its purchases to the payment account.
- Initial release is historical and realized-only; it has no expected or forecast panel.
- Card corrections restate affected installment periods under the implemented Credit Cards specification (`010-credit-cards`); ordinary transaction changes follow Spec 004.
- Evolution grouping matches existing Dashboard 31/93-day boundaries.

## Dependencies and Scope Notes

- Source rules are in Financial Accounts (002), Categories (003), Transactions (004), Transfers (005), Recurring Transactions (006), Dashboard (implemented directory `008-financial-dashboard`), Budgets (`009-monthly-budgets`), Credit Cards (`010-credit-cards`), Financial Goals (`015-financial-goals`), and Recurring Credit Card Purchases (`014-recurring-credit-card-purchases`). Some requested numeric labels differ from existing directory numbering; these paths identify the implemented authority.
- Reports explain what happened financially. Budget planning, goal performance, and forward projections remain separate.
