# Research: Financial Goals

**Date**: 2026-09-25\
**Branch**: `015-financial-goals`\
**Scope**: Resolve technical choices for the clarified [specification](spec.md); all planning choices below have a decision.

## 1. Money and accounting boundary

**Decision**: Store goal amounts in integer BRL centavos, matching Financial Accounts. Keep goal activity in its own domain and never feed it into Transactions, Transfers, financial history, account balance reconcilers, card obligations, Budget consumption, or Dashboard income/expense/result calculations.

**Rationale**: Account `current_balance_centavos` already represents actual holdings. Goal allocation is a designation over that balance, so a balance mutation or normal financial-history entry would double count money. The existing account, transaction, transfer, and card services remain authoritative for actual movements.

**Alternatives considered**: Model each goal as an account, create offsetting transfers, or enter allocations as expenses. Each violates the spec's no-duplicate-money rule. A generic financial transaction subtype would also blur goal history and the financial ledger.

## 2. Goal source of truth and traceability

**Decision**: Persist a goal definition and append-only goal activity. Compute current allocation from accepted initial/add/withdraw events; compute progress, remaining amount, excess, overview totals, and guidance from that value. Preserve account identity at activity time. Record lifecycle and account-association changes as nonmonetary goal events. Use grouped reads and indexes for list/overview performance; do not persist a second asset or reporting balance.

**Rationale**: Activity history explains how a goal reached its current amount and survives account or goal lifecycle changes. The 100-goal/1,000-event acceptance scale does not require an extra persisted allocation total. This avoids divergent state after retries, corrections, or failed writes.

**Alternatives considered**: Store only a mutable `current_allocated` field, which loses audit context; store both an activity sum and cached current amount, which requires reconciliation and adds another mutable source of truth. A cache may be added later only with measured need and invariant tests.

## 3. Capacity, concurrency, and later shortfalls

**Decision**: In a database transaction, lock the goal being changed and all relevant account rows in stable ID order, then recheck ownership/status and aggregate allocations for active/completed goals associated with each account. For creation, lock the candidate account before goal insertion. New linked allocation or funded reassociation is rejected if the resulting designated total exceeds the account's nonnegative current balance. Completing a linked goal also rechecks that the account is active and available with no shortfall; unlinked completion remains possible with an unverified label. Existing transaction/transfer/card payment operations remain free to change actual balance; goal reads then compute and label any negative unallocated amount and block new linked additions. A later shortfall or account archival leaves an already completed goal completed with attention visible.

**Rationale**: Several goals can share one account. Serializing against its row prevents two concurrent goal actions from each claiming the same unallocated amount. Existing financial mutations already lock account rows. A real expense must not be blocked or rewritten to protect a planning designation. Laravel documents [pessimistic row locks](https://laravel.com/docs/13.x/queries#pessimistic-locking) and [transactions with deadlock retry](https://laravel.com/docs/13.x/database#database-transactions).

**Alternatives considered**: Trust client-side capacity, which races; reserve/debit account money, which would turn a goal into a financial movement; automatically shrink goals after spending, which rewrites intent and history; globally lock all user goals, which is unnecessary contention. Do not create a new account balance field for unallocated money.

## 4. Exact-once mutations and conflicts

**Decision**: Require `Idempotency-Key` on goal mutations, following existing transfer, transaction, and card patterns. Scope a key to its owner, persist an operation and request fingerprint with the final response, replay exact retries, and reject a reused key with different action/payload as 409. Goal activity and key result commit in one transaction. Distinct concurrent withdrawal attempts are serialized on the goal row; linked additions are serialized on the account row.

**Rationale**: Double-clicks and uncertain network retries must not add/withdraw twice. Existing project patterns already implement owner-scoped key claims and conflict errors, so no package or novel protocol is needed.

**Alternatives considered**: Frontend-only button disabling, insufficient against retries; a unique timestamp, which does not identify one logical action; reusing a transfer mutation table, which couples separate domains.

## 5. Account lifecycle and historical identity

**Decision**: New or changed links require an owned active Financial Account. Existing account archival does not change goal allocation or activity; goal read projections mark that association inactive and block new additions until unlink/reassociation. Existing app offers no permanent account deletion; retain account ID and a label snapshot on activity, and retain current association label for a degraded unavailable-account display. Account reassociation updates only the goal's declared holding location and records an event; an actual movement still uses Transfers.

**Rationale**: Financial Accounts permits archive/restore, not ordinary hard deletion. Historical goal entries must remain understandable even if an account label changes or cannot load. Credit cards are a separate liability model and never eligible as goal funding accounts.

**Alternatives considered**: Cascading goal deletion on account archive, auto-transfer on reassociation, or counting card credit as savings. All contradict financial integrity or established account lifecycle behavior.

## 6. Read projections, dates, and Dashboard isolation

**Decision**: Goal reads supply authoritative centavos and derived progress/shortfall fields. Calculate monthly guidance from the user's São Paulo business date only when active, underfunded, and future-dated: inclusive calendar-month opportunities, centavo ceiling. Expose an independent Dashboard goals read with at most three active goals, sorted as specified. Keep its loading/error state separate from period-based cash-flow slices.

**Rationale**: The suggestion is a deterministic planning calculation, not a recurring record. A separate Dashboard slice cannot accidentally alter the current total, result, or expected/realized calculations. A compact bounded read protects the dashboard layout and the two-second usability goal.

**Alternatives considered**: Store a monthly projection schedule, reuse recurring transactions, or merge goal values into Dashboard financial totals. Each gives the guidance unsupported financial meaning. Client-side recalculation of money would risk drift from backend rules.

## 7. Frontend architecture and design

**Decision**: Follow the current JavaScript Vue 3 `<script setup>` convention; use a goal API service, a setup-style Pinia store only for shared goal state, and small components with props/emits. Reuse the common currency input, Element Plus form/dialog controls, project tokens, and translations. Add a goal-specific progress component that clamps only the visual track while announcing true overfunding numerically and textually. Follow `docs/design/design-foundation.md`, `app-shell.md`, `navigation.md`, and `components.md` for layout, 44 px controls, 320 px width, 200% zoom, WCAG 2.2 AA, light/dark/system themes, and noncolor cues.

**Rationale**: Existing frontend uses JavaScript, services around `apiRequest`, Pinia setup stores, Vue Router child routes, Element Plus, and an independent Dashboard-card pattern. Official [Vue component guidance](https://vuejs.org/guide/components/props.html) supports explicit props/events; [Pinia setup stores](https://pinia.vuejs.org/core-concepts/) keep shared state/actions central and `storeToRefs` preserves reactivity. The project convention takes precedence over the generic Vue skill's TypeScript default.

**Alternatives considered**: Store all goal data in the Dashboard store, recompute financial values in components, or reuse Budget danger-colored progress semantics. These introduce coupling or misleading status meaning. No extra package is needed.

## Existing integration points checked

- Backend: `routes/api.php`; `app/Models/FinancialAccount.php`; `app/Services/FinancialAccounts/FinancialAccountService.php`; `app/Services/FinancialDashboard/DashboardSummaryService.php`; `app/Services/FinancialDashboard/DashboardAccountsService.php`; `app/Services/FinancialHistory/FinancialHistoryService.php`; `app/Services/Budgets/BudgetCalculationService.php`; transaction, transfer, and card balance reconcilers.
- Frontend: `src/router/index.js`; `src/layouts/AppShell.vue`; `src/components/navigation/AppNavigation.vue`; `src/views/dashboard/FinancialDashboardView.vue`; `src/services/dashboardService.js`; `src/services/httpClient.js`; `src/stores/dashboard/dashboardStore.js`; `src/components/common/CurrencyAmountInput.vue`; `src/components/budgets/BudgetProgressBar.vue`; `src/i18n/messages.js`.
