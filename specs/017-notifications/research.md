# Research: Notifications

**Date**: 2026-09-29 | **Spec**: [spec.md](spec.md)

## 1. Authoritative source integration

**Decision**: Notification evaluators consume existing source services and model states; domain mutations record a durable, owner-scoped projection fact inside their accepted transaction, after authoritative source state is determined. The fact captures any newly qualified type/stage and its effective time, or a request to reconcile prior attention. An evaluator runs after commit, checks current source state to decide active/resolved presentation, and records visible or suppressed identity. Date-driven statement stages are evaluated by the scheduler. A periodic recovery scan covers already-active sources. No notification code computes balances, utilization, goal allocation, statement payment, or recurrence generation.

**Evidence**: `CreditCardObligationReconciler::statusFor` and `CreditCardStatement::outstandingCentavos` own card obligations; `BudgetCalculationService::forMonth` returns exact-centavo `BudgetStatus` for each plan; `FinancialGoalQueryService` projects allocation; `Transaction` and `RecurringCardOccurrence` own review state. Spec 014 and `CardOccurrenceState::isActionable` include reviewable `Failed` occurrences; Notifications includes them in the same occurrence review type without duplicating an earlier Expected or Awaiting over-limit item. Budget threshold alerts are limited to the current business month; prior-month items resolve at rollover, and corrections to ended months remain in Budgets without new alerts.

**Alternatives considered**: Notification-owned financial calculations would drift; read-only polling at center opening fails offline timeliness; broad model observers can fire before domain transactions commit and obscure mutation ownership.

## 2. Durable identity, preferences, and retention

**Decision**: Keep one durable event identity per user/type/source/stage/period-or-target, including suppressed and expired identities. Visible content can be purged after retention, but the minimal key remains to prevent reissue. Record preference changes with effective times; a fact's captured first-qualification time uses the preference effective then, not merely the setting at delayed processing. Serialize preference changes and identity creation per user and enforce a database unique constraint. Source-stage recross reactivates an existing retained item as read; expired identity never reissues.

**Alternatives considered**: Deleting rows at 90 days loses deduplication; checking only current preference would backfill events suppressed while disabled; deduplication by message text breaks when labels or amounts change.

## 3. Timeliness and recovery

**Decision**: Use the existing Laravel scheduler, with a one-minute notification reconciliation command that drains durable projection requests in bounded batches and evaluates date-boundary statement candidates. Run a scheduler process in deployment and the local Docker stack. Evaluate the active user's pending requests on center/count read as a freshness fallback. A bounded recovery sweep reconciles source state and visibility after failures, not a full all-history recalculation on every request. Record lag and failures without logging financial message content.

**Evidence**: `routes/console.php` already schedules `recurring:process-due` at 00:05 America/Sao_Paulo. Current `docker/docker-compose.yml` and test overlay have no scheduler service, so a route-only plan cannot meet the spec's five-minute date boundary target. Application's business timezone is America/Sao_Paulo; `RecurringDateRange` documents no per-user timezone today.

**Alternatives considered**: Browser polling alone excludes offline users; a new push provider or queue package is outside scope; scanning every user's every historical budget on every minute is wasteful. The scheduler and dirty-request approach must be tested against the five-minute target, including midnight and delayed processing.

## 4. Source changes and review destinations

**Decision**: Source mutation integration covers statement/payment/credit/purchase changes, transaction effective/pending/remove/restore/correction changes, recurrence occurrence creation/action, budget plan changes, and goal allocation/target/lifecycle changes. The evaluator uses source-owned authorization and current state. Keep notification destination as a typed, allowlisted context, not an arbitrary stored URL. Resolve it at read/open time.

**Evidence**: Frontend already has statement and goal detail routes, transaction `highlight` query, and recurrence rule `highlight` query. It lacks a direct card-occurrence selection on route entry and a URL-backed budget month/plan highlight. Add those small route behaviors during frontend work. For ordinary recurrence, link to the generated transaction's review in Transactions. For card recurrence, select the exact occurrence, not merely the latest reviewable one.

**Alternatives considered**: Persisting a raw URL risks stale or untrusted navigation; linking only to a recurrence rule can open the wrong occurrence; rendering snapshot content after access loss can leak private data.

## 5. API, storage, and scale

**Decision**: Expose owner-scoped list, unread summary, item/open-read, individual read, mark-all-read, and preference read/update contracts. Cursor pagination uses descending creation time plus identity to keep 10,000-item history stable. An access check or safe tombstone state precedes serialization and unread count. Index by owner/visibility/read/action/time; keep source-identity uniqueness. Return locale-neutral type and context fields plus already safe display content, with one server-owned destination kind and parameters. No HTML in messages.

**Alternatives considered**: Offset pagination degrades as history grows; client-calculated unread count fails on unseen pages; client-constructed URLs can bypass current authorization checks.

## 6. UI and tests

**Decision**: Use existing JavaScript Vue 3 `<script setup>` convention, Pinia for count/center state shared with header, Axios service for requests, and Element Plus/Tailwind semantic tokens. Keep `AppShell` and center route as composition surfaces; split indicator, filter, list/item, and preferences. Use PT-BR/English translations, explicit action/read labels, accessible count, and no automatic non-critical announcements.

**Evidence**: Existing frontend is JavaScript, not TypeScript; `AppHeader.vue` is the global header, and `AppShell.vue` hosts protected routes. Four required design documents live in `docs/design/`.

**Alternatives considered**: A header-only dropdown lacks usable history/preference space; local component state cannot keep header count and center synchronized.

## Resolved technical unknowns

All technical decisions needed for this plan are resolved. Implementation must prove the scheduler is actually running, exact source-state integration covers all listed mutation paths, and contract/retention behavior passes MySQL-backed concurrency and timing checks before frontend starts.
