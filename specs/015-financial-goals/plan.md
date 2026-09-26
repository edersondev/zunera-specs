# Implementation Plan: Financial Goals

**Branch**: `015-financial-goals` | **Date**: 2026-09-25 | **Spec**: [spec.md](spec.md)\
**Input**: Feature specification from `specs/015-financial-goals/spec.md`

**Branch Coordination**: `015-financial-goals` is active in `zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

## Summary

Add an owner-scoped goal allocation layer with traceable allocation, withdrawal, association, and lifecycle history. A goal designates existing money without mutating financial-account balances or financial transactions. Linked allocations are checked against the account's current balance under a lock; later real spending may produce a visible shortfall. Expose goal overview, detail, activity, and a small independent Dashboard section. Complete backend contract, authorization, validation, and tests before frontend work.

## Technical Context

**Language/Version**: PHP 8.3 and Laravel 13.17+; JavaScript and Vue 3.5.40 with `<script setup>`\
**Primary Dependencies**: Existing Sanctum 4, Eloquent, Axios 1.19, Pinia 4, Vue Router 5.2, Element Plus 2.14.5, Tailwind 4.3, vue-i18n 11\
**Storage**: Existing MySQL 8.4 development database; SQLite in-memory tests; integer BRL centavos and owner-scoped goal/activity/mutation records; no account-balance copy\
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, ESLint, Playwright 1.61, OpenAPI contract checks\
**Target Platform**: Authenticated Zunera web application, responsive desktop and mobile\
**Project Type**: Full-stack feature in separate backend and frontend repositories\
**Performance Goals**: Overview and detail usable within 2 seconds for 100 goals and 1,000 activities; bounded Dashboard goals query\
**Constraints**: Exact centavos; America/Sao_Paulo business date; account balance remains authoritative; no new packages, bank connection, automatic transfer, or automatic goal contribution\
**Scale/Scope**: Goal management, activity history, account capacity/shortfall presentation, and compact Dashboard integration; no transaction-to-goal links

## Delivery Scope and Order

1. **Backend** (`../zunera-backend`): Add protected goal contracts, ownership and validation, goal and activity persistence, goal services, atomic capacity and idempotency rules, read projections, Dashboard goals endpoint, and feature/unit tests.
2. **Frontend** (`../zunera-frontend`): Consume verified backend contract in goals routes, service, shared state, accessible views/dialogs, navigation, independent Dashboard card, translations, unit tests, and critical browser journeys.

Frontend implementation starts after the backend contract, authorization, validation, and tests pass. Both application branches must still match this branch at implementation time.

## Constitution Check

*Pre-research gate: PASS on 2026-09-25. Post-design gate: PASS on 2026-09-25.*

- [x] API boundaries use `auth:sanctum` and `session.lifetime`, owner-scoped resolution, Form Requests, API Resources, and safe domain errors. No uploads or new secrets.
- [x] Backend contract, authorization, validation, services, and tests precede frontend implementation; frontend consumes documented shapes and errors.
- [x] The `015-financial-goals` branch matches in specs, backend, and frontend repositories.
- [x] Goal rules live in `App\Services\FinancialGoals`; multi-field inputs use DTOs. Direct Eloquent aggregation suffices, so no repository layer or global service state is planned.
- [x] Vue uses existing JavaScript `<script setup>` convention, service-only API access, Pinia for cross-view goals state, and Element Plus standard controls with project theme tokens.
- [x] Backend feature/unit and contract tests cover money invariants and concurrency; frontend service/store/component tests and isolated Playwright journeys cover critical UI-to-API flows.
- [x] Ownership, untrusted notes, exact money handling, no new packages, and exclusion from existing financial calculations are explicit.
- [x] No constitution exception is required.

Post-design recheck: the contract contains owner-scoped protected reads and
idempotent writes with explicit errors; the data model leaves all existing
financial ledgers and balance reconcilers untouched; the implementation
sequence preserves the backend-first gate; and the quickstart names backend,
frontend, contract, accessibility, and browser verification. All gates remain
passed after Phase 1 design.

## Research and Design Decisions

- [Research](research.md) records money representation, account-capacity locking, append-only activity, idempotency, projection boundaries, and frontend design choices. Laravel 13 and Vue/Pinia documentation was checked for transaction and state patterns.
- [Data model](data-model.md) defines goal, activity, and mutation identity; validation, state transitions, derived progress and capacity, and migration constraints.
- [API contract](contracts/financial-goals-api.yaml) defines owner-scoped goal and Dashboard endpoints, request/response shapes, idempotent mutations, and error semantics.
- [Quickstart](quickstart.md) gives backend-first work order, validation commands, and acceptance walkthrough.

## Implementation Sequence

1. **Backend persistence and boundaries**: Add goal, activity, and mutation-claim storage with ownership, account linkage, status, centavos, and indexes. Define owner-scoped route resolution, Requests, DTOs, Resources, and domain exceptions. Do not alter financial movement tables or balance reconcilers.
2. **Backend creation and money actions**: Implement creation, allocation, and withdrawal in a service. Derive initial account actual/designated/unallocated amounts for the creation contract. Lock relevant account rows before accepting linked money and append dated goal activity atomically. Require idempotency keys for mutations; exact retry replays the prior result and conflicting reuse returns 409. Goal operations never post a transaction or balance effect.
3. **Backend association and coverage**: Implement active-goal metadata edits, unlinking, and reassociation before lifecycle completion. Lock the goal and old/new financial-account rows in a documented order; aggregate active/completed linked allocations and check destination capacity. When later real activity lowers balance, compute and show the exact shortfall. Inactive or unavailable accounts block new additions while retaining readable identity and history.
4. **Backend lifecycle**: Implement explicit complete, reopen, archive, and restore actions after account editing exists. Completion rechecks target allocation and, for a linked goal, active account eligibility and absence of a shortfall; an unlinked goal may complete with unverified backing. Completed allocations still count toward linked-account designation. Preserve activity and reject invalid state transitions without financial side effects.
5. **Backend reads and integration**: Derive current allocation from accepted goal activities, progress, rounded monthly guidance, active-only overview totals, explicit overview attention counts, and a maximum three-goal Dashboard projection. Keep goal activity outside Financial History, Budget calculation, account summaries, Dashboard cash-flow totals, card obligations, and recurrence outputs.
6. **Complete backend verification gate**: Test ownership, validation, cent precision, overfunding, lifecycle, restoration, archived/unlinked accounts, completion blocked during a linked shortfall or inactive/unavailable association, completed status preserved after a later shortfall or account archival, account capacity under overlapping requests, idempotent replay, one-to-many account goals, due-date guidance, and unchanged financial calculations. Verify the full backend suite, style, contract, and migration on SQLite tests and the MySQL development stack before any frontend work.
7. **Frontend feature**: Add goal overview/detail/archive routes and navigation; goal service and Pinia state; form and allocation/withdrawal dialogs; progress, activity, account-capacity and attention views. Reuse currency/date utilities and Element Plus controls. Keep service errors actionable and refresh authoritative values after mutations.
8. **Frontend Dashboard and verification**: Add a separate goals section with independent loading/error/retry state and no coupling to period-sensitive financial totals. Add PT-BR/English copy, keyboard and screen-reader semantics, responsive/light/dark/system checks, safe rendering of user names/notes, service/store/component tests, and Playwright create–allocate–withdraw–complete–reopen–archive–restore and shortfall journeys. Finish with uncoached SC-001 and SC-004 usability checks and recorded results.

## Project Structure

### Documentation (this feature)

```text
specs/015-financial-goals/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/financial-goals-api.yaml
└── checklists/requirements.md
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/Models/FinancialGoal.php
├── app/Models/FinancialGoalActivity.php
├── app/Http/Controllers/Api/V1/FinancialGoalController.php
├── app/Http/Controllers/Api/V1/FinancialGoalDashboardController.php
├── app/Http/Requests/FinancialGoals/
├── app/Http/Resources/FinancialGoals/
├── app/Data/FinancialGoals/
├── app/Services/FinancialGoals/
├── database/migrations/
├── routes/api.php
└── tests/{Feature,Unit}/FinancialGoals/

../zunera-frontend/
├── src/router/index.js
├── src/layouts/AppShell.vue
├── src/components/navigation/AppNavigation.vue
├── src/views/goals/
├── src/components/goals/
├── src/components/dashboard/DashboardGoalsCard.vue
├── src/services/financialGoalService.js
├── src/stores/goals/financialGoalStore.js
├── src/i18n/messages.js
├── src/utils/goals/
└── e2e/financial-goals.spec.js
```

**Structure Decision**: Match existing domain folders and API versioning. Use project JavaScript convention despite the Vue skill's general TypeScript default. Small route views compose feature components; Pinia owns only state shared by overview, detail, and Dashboard. Exact file names may be refined during task breakdown without changing these boundaries.

## Frontend Component Map

| Component | Single responsibility | Inputs / outputs |
|---|---|---|
| GoalOverviewView | Compose summary, status sections, and empty/error states | Reads store; opens actions and routes to detail |
| GoalDetailView | Compose progress, account context, guidance, history, and actions | Goal ID input; dispatches store actions |
| GoalCard | Summarize one goal in a list | Goal prop; detail/action events |
| GoalProgress | Show true numeric progress with visually clamped track | Current/target props; accessible text output |
| GoalFormDialog | Gather name, target, date, notes, account, initial amount | Initial values and account choices; submit/cancel events |
| GoalAmountDialog | Gather one allocation or withdrawal | Goal/action/capacity props; submit/cancel events |
| GoalActivityList | Show paged, goal-only activity | Activity prop/page; page-change event |
| DashboardGoalsCard | Show three selected goals independently of financial totals | Goals/loading/error props; retry/navigation events |
