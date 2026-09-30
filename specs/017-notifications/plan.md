# Implementation Plan: Notifications

**Branch**: `017-notifications` | **Date**: 2026-09-29 | **Spec**: [spec.md](spec.md)  
**Input**: Clarified Notifications specification in `specs/017-notifications/spec.md`

**Branch Coordination**: `017-notifications` is active in specs, `../zunera-backend`, and `../zunera-frontend`. Verify before editing either app. The user's “Spec 012” label conflicts with an existing directory; `017-notifications` is the active feature directory.

## Summary

Deliver an owner-scoped in-app Notification Center and header unread indicator for statement deadlines, recurrence review, category-plan thresholds, and goal milestones. A durable notification identity records each meaningful source stage once, including suppressed/expired events. Source-domain projections supply all financial state; Notifications stores awareness, not another ledger. Source mutation requests plus a running minute scheduler provide timely evaluation and recovery. Backend contract, authorization, validation, and tests complete before Vue integration.

## Technical Context

**Language/Version**: PHP 8.3; Laravel `^13.17`; JavaScript with Vue `^3.5.40` and `<script setup>` (existing convention)  
**Primary Dependencies**: Existing Sanctum, Eloquent/query builder, Carbon, Axios `^1.19`, Pinia `^4.0.2`, Vue Router `^5.2.0`, Element Plus `^2.14.5`, Tailwind `^4.3`, vue-i18n; no new package  
**Storage**: Existing MySQL application database; SQLite for fast tests; new notification identity, preference history, and projection-request records; no copied financial ledger  
**Testing**: Laravel feature/unit/API-contract and MySQL concurrency/timing tests; Pint; Vitest service/store/component tests; isolated Playwright UI-to-API journeys; lint/build  
**Target Platform**: Authenticated responsive Zunera web app, personal `user_id` ownership, PT-BR/English display, America/Sao_Paulo financial business date, Light/Dark/System themes  
**Project Type**: Full-stack feature across separate backend and frontend repositories  
**Performance Goals**: With application, database, and evaluator healthy, at least 19 of 20 qualifying source-change/date-boundary events visible in an authorized response within five minutes of commit/boundary; 95% of first-page/filter loads with 10,000 retained items within three seconds; no duplicate equivalent event  
**Constraints**: Current source-domain calculations authoritative; no future channels/packages/uploads/secrets; no raw full card number; current access on every list/count/open; unresolved action retained; expired identity remains for dedup  
**Scale/Scope**: Eight semantic notification types across four preference categories; one header summary, three center views, 25-item default cursor page, four preferences; no activity feed or permanent notification audit

## Delivery Scope and Order

1. **Backend** (`../zunera-backend`): Add protected API contract, Form Requests, Resources, source evaluators, service-owned lifecycle, persistence, event identity, preference history, date scheduler and runtime, authorization, and tests. Complete all six stories' backend contracts, authorization, validation, services, tests, and cross-cutting backend verification before any frontend implementation.
2. **Frontend** (`../zunera-frontend`): Consume verified API through Axios service and shared Pinia state; add header indicator, center, preferences, typed navigation, locales, and tests. Keep financial computations on server.

## Constitution Check

*Pre-research gate: PASS on 2026-09-29. Post-design gate: PASS on 2026-09-29.*

- [x] Protected `auth:sanctum` plus `session.lifetime` routes, Form Requests for query/mutation validation, API Resources for data; no upload path.
- [x] Backend contract, authorization, validation, services, and tests precede frontend implementation; frontend consumes `contracts/notifications-api.yaml`.
- [x] All three repositories use `017-notifications` branch.
- [x] Service layer owns event evaluation/lifecycle. Input DTOs only for multi-field projection context; a read repository is justified for indexed owner-scoped list/count and cross-source candidate selection, not simple CRUD.
- [x] Vue Composition API with `<script setup>`, existing JavaScript convention, Axios services, Pinia only for header/center shared state, Element Plus and semantic Tailwind styles.
- [x] API feature/contract tests, domain transition/unit tests, real MySQL duplicate/concurrency checks, and isolated Playwright journeys.
- [x] Owner/source authorization, safe text, no full card data, preference history, retention, runtime scheduler, failure metrics, and no new secrets/packages documented.
- [x] No constitution exception. Complexity Tracking omitted.

## Phase 0: Research

See [research.md](research.md). Key decisions:

1. Consume `CreditCardObligationReconciler`, `BudgetCalculationService`, goal projection, and recurrence/transaction source states. No notification-side math.
2. Use durable event identities and suppressed/expired tombstones, plus preference effective-time history; unique owner/event key handles retries and concurrency.
3. Use durable projection requests at source mutation boundaries and a one-minute scheduled evaluator with recovery and current-user freshness fallback. Add an actually running scheduler process to deployment/local Docker.
4. Resolve typed destinations against current source authorization. Add exact card-occurrence and budget-month/plan route handling rather than arbitrary URLs.

## Phase 1: Design and Contracts

See [data-model.md](data-model.md) for event, preference/history, and projection-request state. See [contracts/notifications-api.yaml](contracts/notifications-api.yaml) for the owner-scoped HTTP contract. See [quickstart.md](quickstart.md) for implementation and verification sequence.

### Backend behavior and source boundary

- Introduce a Notifications service that converts authoritative source projections into candidate event keys and current action state. Place candidate lookup and indexed history/count access behind a focused query service/repository only where cross-source or high-volume queries justify it.
- Record durable projection facts inside accepted source transactions after authoritative state is determined; process them after commit. Capture newly qualified type/stage and effective time from the source projection, plus reconciliation needs for old attention, so delayed processing cannot misclassify disabled-interval events or lose a brief goal milestone. Cover credit-card statement changes, payments and credit events; transaction pending/effective/remove/restore and date/category/amount corrections; recurrence occurrence generation and review actions; budget plan creation/change/removal/copy; and goal allocation, target, and lifecycle changes. Do not trigger for ordinary routine operations that never reach matrix criteria.
- Time-based evaluation must use `RecurringDateRange::BUSINESS_TIMEZONE` and the Credit Cards reconciler before assessing statement status. Run notification reconciliation each minute, drain requests in bounded owner batches, evaluate due-stage candidates, and retry failures. Run recovery scans for active/stale items and missed source requests. Make scheduler runtime explicit in Docker and deployment; verify it executes, not merely registers a command.
- Budget evaluation calls `BudgetCalculationService::forMonth` for the current business month and reads each plan's `BudgetStatus`. A correction to an ended month updates Budgets but cannot generate a new alert. At the next business-month boundary, resolve prior-month active budget items and retain their 90-day history. One jump emits only highest stage and consumes skipped lower identities. Projected status is ignored. Repeated crossing within the current month reactivates a retained emitted item as read; expired identity is never reissued.
- Goal evaluation uses authoritative current allocation/target and active status; key includes exact target centavos. Recurrence review keys use actual generated transaction or card-occurrence ID. Expected, awaiting-over-limit, and reviewable failed card occurrence states share one attention item; update its safe review/retry context when source reason changes, and resolve it when recorded or dismissed. Statement stage follows source status and positive outstanding; full payment resolves, payment reversal can reactivate retained current stage.
- Preference writes and event creation must be atomic with respect to per-user choice, using effective-time history. Store a suppressed identity when first qualification happens during a disabled interval, preventing backfill after re-enable.
- Serialize only authorized content. Missing/deleted source yields safe historical item without active destination when owner provenance remains valid; changed ownership/access excludes it from list/count/open. All item mutations use owner scope and source authorization, returning indistinguishable 404 for unknown/forbidden.
- Mark-all-read affects every currently accessible unread item as of action, not just loaded page. Open and explicit read leave source action unchanged. Resolve from domain state, never from user read. Expire content at 90 calendar days after later creation/resolution, retaining minimal identity. Instrument evaluator lag, failures, duplicate-key contention, and scheduler heartbeat without financial text.

### Frontend component map

| Surface | Responsibility | Inputs / outputs |
| --- | --- | --- |
| `AppShell` | Compose protected routes and shared notification state | Session/store to header and routed center |
| `AppHeader` / `NotificationIndicator` | Show accessible count and open center | Exact count in; navigation event out; display none/1/2–99/99+ |
| `NotificationsView` | Route-level center composition | View query and store state to children; coordinates refresh |
| `NotificationFilterBar` | Choose All, Unread, Requires action | Selected view in; change event out |
| `NotificationList` / `NotificationItem` | Paged chronological items with semantic/read/action state | Items in; open/read/load-more events out |
| `NotificationPreferences` | Four broad category toggles | Preferences in; category change event out |
| `notificationService.js` | Axios calls and typed contract normalization | Validated request inputs; API response/errors |
| `notificationStore.js` | Shared summary, pages, preferences, loading/errors, session reset | Explicit fetch/open/read/update actions |

Use existing `<script setup>` JavaScript and props/events. Route view stays a composition surface; services hold transport; store owns only shared data; computed derives count labels and visible state. Add route destination map for statement detail, highlighted transaction, exact card recurrence occurrence, selected budget month/plan, and goal detail. Preserve focus after navigation, use plain escaped text, localized date/money, and existing semantic tokens. Clear store on logout/account switch; no cross-session count flash. Refresh count on shell entry, center mutations, focus/visibility return, and a bounded interval while active so the header stays current without push.

### Verification gates and risks

- **Financial integrity**: Compare source balances, statement obligations, budget utilization, goal allocation, and recurring status before/after notification-only actions.
- **Timeliness**: A scheduler worker must actually run. Existing local Docker setup has no scheduler container; add it during backend implementation and document production process. Simulate midnight, late first eligibility, processing failure/retry, and recurrence processor at 00:05.
- **Preference races**: Test event first qualifies while disabled, then user enables before delayed evaluation. Test concurrent preference update and identical event evaluation in MySQL.
- **Source changes**: Audit every relevant mutation path and route destination. Payment reversals, historical budget corrections (no new alerts for ended months), month-boundary resolution, source deletion, and authorization loss are high-risk regressions.
- **History scale**: Load 10,000 retained items; check owner-scoped cursor and count indexes, bounded serialization, stable equal-time ordering, and three-second first page target.
- **Accessibility**: Test keyboard, screen reader labels/status, 320 px and 200% zoom, light/dark/system, English/PT-BR, and no background announcement flood.

## Project Structure

### Documentation (this feature)

```text
specs/017-notifications/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/notifications-api.yaml
└── checklists/requirements.md
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/Services/Notifications/
├── app/Http/Controllers/Api/V1/NotificationController.php
├── app/Http/Controllers/Api/V1/NotificationPreferenceController.php
├── app/Http/Requests/Notifications/
├── app/Http/Resources/Notifications/
├── app/Models/NotificationEvent.php
├── app/Models/NotificationPreference.php
├── app/Console/Commands/
├── database/migrations/
├── routes/api.php
├── routes/console.php
├── docker/
└── tests/Feature/Notifications/ and tests/Unit/Notifications/

../zunera-frontend/
├── src/layouts/AppShell.vue
├── src/components/layout/AppHeader.vue
├── src/components/notifications/
├── src/views/notifications/NotificationsView.vue
├── src/services/notificationService.js
├── src/stores/notifications/notificationStore.js
├── src/router/index.js
├── src/i18n/
└── e2e/notifications.spec.js
```

**Structure Decision**: Keep existing repository layout and JavaScript Vue convention. Additional persistence models and command files follow the design, without a new library or independent financial model. Backend contract and tests gate frontend work.
