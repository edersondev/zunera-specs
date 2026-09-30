# Tasks: Notifications

**Input**: [spec.md](spec.md), [plan.md](plan.md), [research.md](research.md), [data-model.md](data-model.md), [API contract](contracts/notifications-api.yaml), [quickstart.md](quickstart.md)
**Tests**: Required by SQR-002, the constitution, and the plan. Write focused tests before or with implementation; run them at each backend gate.
**Organization**: All six backend story increments and feature-wide backend verification precede every frontend task. Story labels connect backend and frontend increments. No new package.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm branches and map authoritative source mutations before application edits.

- [X] T001 Verify `017-notifications` is active in specs, `../zunera-backend`, and `../zunera-frontend`; record the check in `specs/017-notifications/checklists/branch-coordination.md`.
- [X] T002 Map all accepted statement, card-credit, purchase, transaction, recurrence, budget-plan, and goal mutation entry points to notification projection facts in `specs/017-notifications/checklists/source-mutation-audit.md`, using `../zunera-backend/app/Services/` as evidence.
- [X] T003 [P] Record exact API examples and source-to-destination cases for implementation in `specs/017-notifications/contracts/fixtures/notifications.yaml`, matching `specs/017-notifications/contracts/notifications-api.yaml`.

**Checkpoint**: Branches, source inventory, and fixtures ready.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish durable identity, owner authorization, preference timing, and running evaluation.

- [X] T004 Add migration for durable event keys, read/resolved/visibility state, safe snapshots, and owner/count/cursor indexes in `../zunera-backend/database/migrations/2026_09_29_000001_create_notification_events_table.php`.
- [X] T005 [P] Add migrations for current preferences and effective-time changes in `../zunera-backend/database/migrations/2026_09_29_000002_create_notification_preferences_tables.php`.
- [X] T006 [P] Add migration for durable source-verified projection facts with retry/lag fields in `../zunera-backend/database/migrations/2026_09_29_000003_create_notification_projection_facts_table.php`.
- [X] T007 Create `../zunera-backend/app/Models/NotificationEvent.php`, `../zunera-backend/app/Models/NotificationPreference.php`, `../zunera-backend/app/Models/NotificationPreferenceChange.php`, and `../zunera-backend/app/Models/NotificationProjectionFact.php`; preserve suppressed/expired event identity and avoid source-entity cascade deletion.
- [X] T008 [P] Add unit tests for event-key uniqueness, stage identity, skipped budget-stage consumption, and retained/expired recross behavior in `../zunera-backend/tests/Unit/Notifications/NotificationIdentityTest.php`.
- [X] T009 Implement event-key builder and idempotent identity/visibility transitions with database uniqueness in `../zunera-backend/app/Services/Notifications/NotificationEventService.php`; make T008 pass under retries.
- [X] T010 [P] Add unit tests for preference effective-time lookup and disabled-then-re-enabled delayed processing in `../zunera-backend/tests/Unit/Notifications/NotificationPreferenceTimelineTest.php`.
- [X] T011 Implement default-on four-category preference timeline and serialized owner/category updates in `../zunera-backend/app/Services/Notifications/NotificationPreferenceService.php`; make T010 pass.
- [X] T012 [P] Add feature tests for owner/source authorization, safe missing-source history, and typed destinations in `../zunera-backend/tests/Feature/Notifications/NotificationAccessTest.php`.
- [X] T013 Implement source authorization and typed destination resolver in `../zunera-backend/app/Services/Notifications/NotificationSourceResolver.php`; exclude newly unauthorized items from list/count/open and return no raw URLs.
- [X] T014 [P] Add feature tests for durable fact capture, duplicate drain, retry after failure, and stale source state in `../zunera-backend/tests/Feature/Notifications/NotificationProjectionFactTest.php`.
- [X] T015 Implement transaction-safe source fact writer and bounded evaluator in `../zunera-backend/app/Services/Notifications/NotificationProjectionFactService.php` and `../zunera-backend/app/Services/Notifications/NotificationReconciler.php`; capture source-qualified stage/time inside accepted domain transactions and consume after commit.
- [X] T016 Add one-minute reconciliation command, retry/retention hooks, and scheduler heartbeat in `../zunera-backend/app/Console/Commands/ReconcileNotifications.php` and `../zunera-backend/routes/console.php`; cover command behavior in `../zunera-backend/tests/Feature/Notifications/NotificationSchedulerTest.php`.
- [X] T017 Configure an actually running scheduler process in `../zunera-backend/docker/docker-compose.yml` and `../zunera-backend/docker/docker-compose.test.yml`; document production invocation and heartbeat check in `specs/017-notifications/quickstart.md`.
- [X] T018 Run migration and foundational notification tests on SQLite and upgraded MySQL, plus Pint, then record the backend foundation gate in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: Foundation verified; no frontend work.

---

## Phase 3: Backend US1 - Notification Center (P1)

**Goal**: Owner-scoped center, summary, read/open API, and seeded-history behavior.

**Independent Test**: Seed mixed owner/foreign items and verify count, pages, filters, safe opening, and no financial mutation.

- [X] T019 [P] [US1] Write list/summary/open/read/read-all API contract and validation tests, including PT-BR/English notification text and missing/unsupported `Accept-Language` fallback, from `specs/017-notifications/contracts/notifications-api.yaml` in `../zunera-backend/tests/Feature/Notifications/NotificationCenterContractTest.php`.
- [X] T020 [P] [US1] Write retention, cursor stability with equal timestamps, 99/100 unread, filter, read-versus-resolved, and bounded owner-pending-fact freshness on list/summary reads in `../zunera-backend/tests/Feature/Notifications/NotificationCenterLifecycleTest.php`.
- [X] T021 [US1] Implement owner-scoped indexed list/count, cursor, retention, bulk read, individual read, and open behavior in `../zunera-backend/app/Services/Notifications/NotificationCenterService.php`; before list/summary responses, boundedly evaluate that owner's pending source facts when scheduler processing lags, and check current authorization for every visible item.
- [X] T022 [US1] Add validated list/filter/cursor query in `../zunera-backend/app/Http/Requests/Notifications/ListNotificationsRequest.php` and safe plain-text `../zunera-backend/app/Http/Resources/Notifications/NotificationResource.php`; validate item identity on read/open and match all contract response shapes.
- [X] T023 [US1] Add protected list, summary, read, read-all, and open routes/controller in `../zunera-backend/routes/api.php` and `../zunera-backend/app/Http/Controllers/Api/V1/NotificationController.php`; use existing auth/session middleware and indistinguishable 404 for foreign/expired items.
- [X] T024 [US1] Run T019–T023 backend tests plus authorization/validation regressions and record verified API contract and financial-integrity gate in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: Story-specific backend contract, source integration, and tests pass.

---

## Phase 4: Backend US2 - Card Statement Deadlines (P1)

**Goal**: Source-authoritative approaching, due-today, and overdue stages.

**Independent Test**: Advance a partially paid statement through stages and payment/reversal; confirm one item per stage and correct resolution.

- [X] T025 [P] [US2] Write statement notification tests for 1–3-day first eligibility, due-today, overdue, partial/full payment, credit application, payment reversal, zero-balance/open exclusion, Sao Paulo midnight, and a changed presentation timezone (or user timezone where supported) that must not reissue the same business event in `../zunera-backend/tests/Feature/Notifications/StatementNotificationTest.php`.
- [X] T026 [US2] Implement authoritative statement stage projection using `CreditCardObligationReconciler` and masked card identity in `../zunera-backend/app/Services/Notifications/StatementNotificationProjector.php`; never derive payment independently.
- [X] T027 [US2] Integrate transactional projection facts at the T002-audited mutation points in `../zunera-backend/app/Services/CreditCards/CreditCardStatementService.php`, `../zunera-backend/app/Services/CreditCards/CreditCardStatementPaymentService.php`, `../zunera-backend/app/Services/CreditCards/CreditCardCreditEventService.php`, `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php`, and `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseCorrectionService.php`; preserve source financial calculations.
- [X] T028 [US2] Add minute date-stage candidate scan and stale-stage/payment resolution to `../zunera-backend/app/Console/Commands/ReconcileNotifications.php` and `../zunera-backend/app/Services/Notifications/NotificationReconciler.php`; prove offline due-boundary delivery with T025.
- [X] T029 [US2] Run statement notification, Credit Cards, and API authorization tests plus MySQL duplicate evaluation; record backend gate and observed scheduler lag in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: Story-specific backend contract, source integration, and tests pass.

---

## Phase 5: Backend US3 - Recurrence Review (P1)

**Goal**: One review item per ordinary/card occurrence with exact source state.

**Independent Test**: Generate and resolve both occurrence kinds; check Expected→Failed stays one item and exact destination.

- [X] T030 [P] [US3] Write recurrence review tests for ordinary pending/effective/removed and card expected/over-limit/failed/recorded/dismissed, paused rule, duplicate attempts, and no issuer-charge claim in `../zunera-backend/tests/Feature/Notifications/RecurringReviewNotificationTest.php`.
- [X] T031 [US3] Implement ordinary/card source-state adapters with one review event key per generated occurrence and safe failure/retry summary in `../zunera-backend/app/Services/Notifications/RecurringReviewProjector.php`.
- [X] T032 [US3] Record durable projection facts on occurrence generation/action and generated transaction effective/remove/restore paths in `../zunera-backend/app/Services/RecurringTransactions/RecurringOccurrenceService.php`, `../zunera-backend/app/Services/RecurringTransactions/RecurringCardOccurrenceActionService.php`, and `../zunera-backend/app/Services/Transactions/TransactionService.php`; use T002 audit for any additional accepted path, and do not confirm or record a purchase from Notifications.
- [X] T033 [US3] Run recurrence, transaction, API-contract, and notification tests; record backend gate including Expected→Failed same-item assertion in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: Story-specific backend contract, source integration, and tests pass.

---

## Phase 6: Backend US4 - Budget Thresholds (P2)

**Goal**: Current-month category-plan stages from Budgets realized status.

**Independent Test**: Cross thresholds, jump directly to exceeded, fall/recross, and roll month; compare to source status.

- [X] T034 [P] [US4] Write budget tests for 80/reached/exceeded, direct jump, skipped stages, repeated crossing, projected-only amount, monthly total exclusion, ended-month correction, and rollover resolution in `../zunera-backend/tests/Feature/Notifications/BudgetThresholdNotificationTest.php`.
- [X] T035 [US4] Implement current-month category-plan projection from `BudgetCalculationService::forMonth` and skipped-stage identity consumption in `../zunera-backend/app/Services/Notifications/BudgetNotificationProjector.php`; never recalculate utilization.
- [X] T036 [US4] Add affected-month fact capture to `../zunera-backend/app/Services/Budgets/BudgetService.php`, `../zunera-backend/app/Services/Transactions/TransactionService.php`, `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php`, `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseCorrectionService.php`, and `../zunera-backend/app/Services/CreditCards/CreditCardCreditEventService.php`; use T002 audit for any other recognized-spend path without altering source totals.
- [X] T037 [US4] Add business-month rollover resolution to `../zunera-backend/app/Services/Notifications/NotificationReconciler.php` and scheduler test coverage in `../zunera-backend/tests/Feature/Notifications/NotificationSchedulerTest.php`.
- [X] T038 [US4] Run budget/card/transaction and notification tests; record exact source reconciliation and backend gate in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: Story-specific backend contract, source integration, and tests pass.

---

## Phase 7: Backend US5 - Goal Milestone (P2)

**Goal**: One target-state milestone without completing the goal.

**Independent Test**: Reach, fall, regain, and change target; confirm keys and unchanged goal/account state.

- [X] T039 [P] [US5] Write goal milestone tests for allocation/target change, same-target regain, different target, archived/completed exclusion, and unchanged balances/status in `../zunera-backend/tests/Feature/Notifications/GoalMilestoneNotificationTest.php`.
- [X] T040 [US5] Implement source-projected active-goal target qualification and target-state event identity in `../zunera-backend/app/Services/Notifications/GoalMilestoneProjector.php`.
- [X] T041 [US5] Capture qualification facts after accepted goal allocation, withdrawal, target edit, and lifecycle operations in `../zunera-backend/app/Services/FinancialGoals/FinancialGoalMutationService.php`; retain brief reached milestone through delayed processing.
- [X] T042 [US5] Run goal/notification financial-integrity and API tests; record backend gate in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: Story-specific backend contract, source integration, and tests pass.

---

## Phase 8: Backend US6 - Category Preferences (P2)

**Goal**: Four future-only optional categories with effective-time suppression.

**Independent Test**: Disable and re-enable a category around qualifying events; verify old identity stays suppressed and new event appears.

- [X] T043 [P] [US6] Write preference GET/PATCH contract, invalid category/body, owner scope, concurrent toggle/event, suppressed identity, delayed evaluation, and Critical opt-out tests in `../zunera-backend/tests/Feature/Notifications/NotificationPreferenceContractTest.php`.
- [X] T044 [US6] Implement preference Form Request, Resource, controller, and protected routes in `../zunera-backend/app/Http/Requests/Notifications/UpdateNotificationPreferenceRequest.php`, `../zunera-backend/app/Http/Resources/Notifications/NotificationPreferenceResource.php`, `../zunera-backend/app/Http/Controllers/Api/V1/NotificationPreferenceController.php`, and `../zunera-backend/routes/api.php`; use foundational timeline service.
- [X] T045 [US6] Verify first-qualification preference timing with overlapping MySQL requests and all matrix sources in `../zunera-backend/tests/Feature/Notifications/NotificationPreferenceConcurrencyTest.php`; run backend suite and record gate in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: Story-specific backend contract, source integration, and tests pass.

---

## Phase 9: Feature-Wide Backend Verification Gate

**Purpose**: Complete backend concurrency, privacy, timeliness, exclusion, and full-suite checks before frontend work.

- [X] T046 [P] Add real MySQL simultaneous evaluation, preference race, and 10,000-event owner/cursor/count performance coverage in `../zunera-backend/tests/Feature/Notifications/NotificationConcurrencyAndScaleTest.php`.
- [X] T047 [P] Add tests for deleted/archived/unauthorized source, event snapshot redaction, 90-day expiry, stale destination, and foreign-ID non-disclosure in `../zunera-backend/tests/Feature/Notifications/NotificationPrivacyRetentionTest.php`.
- [X] T048 With healthy app, MySQL, and evaluator in the local Docker stack, measure at least 20 qualifying source-change and business-day-boundary cases, including recurrence processor timing; record each commit/boundary and first authorized in-app response in `specs/017-notifications/checklists/timeliness-performance.md`, require at least 95% within five minutes, and separately test injected failure/recovery for eventual delivery without duplicates.
- [X] T049 Add backend negative acceptance cases for non-qualifying transaction creation/edit, transfer, successful statement payment, automatic recurrence generation, goal contribution, report generation, and budget edit in `../zunera-backend/tests/Feature/Notifications/NotificationExclusionTest.php`; then run the complete backend contract/domain/concurrency/security suite plus Pint and record the feature-wide backend gate in `specs/017-notifications/checklists/backend-gates.md`.

**Checkpoint**: T049 records a passing feature-wide backend gate. Frontend may start only now.

---

## Phase 10: Frontend US1 - Notification Center (P1)

**Goal**: Global indicator, center, filters, read/open, and safe navigation against verified API.

**Independent Test**: With seeded API items, verify header, ordering, filters, navigation, read actions, and empty states.

- [X] T050 [P] [US1] Write service and shared-store tests for list/summary/open/read/read-all, 99+ label, stale session reset, and page/filter state in `../zunera-frontend/src/services/__tests__/notificationService.spec.js` and `../zunera-frontend/src/stores/notifications/__tests__/notificationStore.spec.js`.
- [X] T051 [US1] Implement Axios transport and shared Pinia summary/center actions in `../zunera-frontend/src/services/notificationService.js` and `../zunera-frontend/src/stores/notifications/notificationStore.js`; use only verified backend responses.
- [X] T052 [US1] Add accessible `../zunera-frontend/src/components/notifications/NotificationIndicator.vue` to `../zunera-frontend/src/components/layout/AppHeader.vue`; in `../zunera-frontend/src/layouts/AppShell.vue`, refresh count on shell entry, center mutations, focus/visibility return, and a bounded interval while active, then clear it on session switch; no unread count at zero, exact through 99, then 99+.
- [X] T053 [US1] Implement protected center route and thin view in `../zunera-frontend/src/router/index.js` and `../zunera-frontend/src/views/notifications/NotificationsView.vue`; compose paged All/Unread/Requires action controls from `../zunera-frontend/src/components/notifications/NotificationFilterBar.vue`, `../zunera-frontend/src/components/notifications/NotificationList.vue`, and `../zunera-frontend/src/components/notifications/NotificationItem.vue`.
- [X] T054 [US1] Add allowlisted destination mapping with safe unavailable-source fallback in `../zunera-frontend/src/services/notificationDestination.js`; keep title, read, pending, and resolved meaning available without color.
- [X] T055 [US1] Add PT-BR/English center, count, empty/filter, read/action, and unavailable copy in `../zunera-frontend/src/i18n/messages.js`; format server text/date/BRL using existing conventions.
- [X] T056 [US1] Add component and isolated UI-to-API journey tests for indicator, shell-entry/focus/visibility/interval refresh, session reset, paging, open/read, mark-all, keyboard focus, empty history, and read-but-unresolved state in `../zunera-frontend/src/components/notifications/__tests__/NotificationCenter.spec.js` and `../zunera-frontend/e2e/notifications.spec.js`; run unit/lint/build/this Playwright file.

**Checkpoint**: Story-specific UI and journey tests pass against the established backend contract.

---

## Phase 11: Frontend US2 - Card Statement Deadlines (P1)

**Goal**: Masked card context and statement destination.

**Independent Test**: Open each statement stage and verify masked text, current action, and read without payment.

- [X] T057 [US2] Map `credit_card_statement` destination to the existing statement detail route and show source-owned masked card/amount text in `../zunera-frontend/src/services/notificationDestination.js` and `../zunera-frontend/src/components/notifications/NotificationItem.vue`.
- [X] T058 [US2] Add service/component/Playwright coverage for upcoming→due today→overdue, partial/full payment, no full card number, and read-without-payment in `../zunera-frontend/src/components/notifications/__tests__/StatementNotification.spec.js` and `../zunera-frontend/e2e/notifications.spec.js`.

**Checkpoint**: Story-specific UI and journey tests pass against the established backend contract.

---

## Phase 12: Frontend US3 - Recurrence Review (P1)

**Goal**: Exact occurrence review destinations and changed failure context.

**Independent Test**: Open ordinary/card items and verify exact occurrence, retry context, and source-side resolution.

- [X] T059 [US3] Add exact `occurrence_id` deep-link handling to `../zunera-frontend/src/views/recurring-transactions/RecurringTransactionsListView.vue` and map generated transaction highlight/card occurrence destination in `../zunera-frontend/src/services/notificationDestination.js`; never substitute the newest different occurrence.
- [X] T060 [US3] Add component and isolated Playwright coverage for ordinary/card review navigation, failed retry context, read without confirmation, and external source resolution in `../zunera-frontend/src/views/recurring-transactions/__tests__/NotificationOccurrenceLink.spec.js` and `../zunera-frontend/e2e/notifications.spec.js`.

**Checkpoint**: Story-specific UI and journey tests pass against the established backend contract.

---

## Phase 13: Frontend US4 - Budget Thresholds (P2)

**Goal**: Plan/month navigation and threshold meaning.

**Independent Test**: Open category threshold item and verify plan/month, highest-stage jump, and old-month resolution.

- [X] T061 [US4] Add URL-backed year/month and plan highlight handling to `../zunera-frontend/src/views/budgets/BudgetsView.vue` and typed `budget_plan` mapping in `../zunera-frontend/src/services/notificationDestination.js`.
- [X] T062 [US4] Add component and Playwright coverage for category threshold text, destination month/plan, highest-stage jump, and resolved previous-month attention in `../zunera-frontend/src/views/budgets/__tests__/BudgetNotificationLink.spec.js` and `../zunera-frontend/e2e/notifications.spec.js`.

**Checkpoint**: Story-specific UI and journey tests pass against the established backend contract.

---

## Phase 14: Frontend US5 - Goal Milestone (P2)

**Goal**: Informational goal milestone and goal detail navigation.

**Independent Test**: Open milestone and verify goal destination, no pending badge, and no completion mutation.

- [X] T063 [US5] Map goal destination and present informational success without an action-pending badge in `../zunera-frontend/src/services/notificationDestination.js` and `../zunera-frontend/src/components/notifications/NotificationItem.vue`.
- [X] T064 [US5] Add component and Playwright coverage for one milestone, goal detail navigation, same-target regain, and no automatic completion in `../zunera-frontend/src/components/notifications/__tests__/GoalMilestoneNotification.spec.js` and `../zunera-frontend/e2e/notifications.spec.js`.

**Checkpoint**: Story-specific UI and journey tests pass against the established backend contract.

---

## Phase 15: Frontend US6 - Category Preferences (P2)

**Goal**: Accessible toggles for four categories with preserved history.

**Independent Test**: Toggle category off/on; verify future-only items, Critical opt-out, and retained old history.

- [X] T065 [P] [US6] Write service/store tests for four defaults, toggles, server errors, and history preservation in `../zunera-frontend/src/services/__tests__/notificationPreferenceService.spec.js` and `../zunera-frontend/src/stores/notifications/__tests__/notificationPreferenceStore.spec.js`.
- [X] T066 [US6] Add preference transport/store actions in `../zunera-frontend/src/services/notificationService.js` and `../zunera-frontend/src/stores/notifications/notificationStore.js`; do not clear historical items when toggled.
- [X] T067 [US6] Build accessible four-category controls inside center using `../zunera-frontend/src/components/notifications/NotificationPreferences.vue` and `../zunera-frontend/src/views/notifications/NotificationsView.vue`; say Critical follows its category choice.
- [X] T068 [US6] Add component and Playwright cases for opt-out, re-enable without backfill, preserved old history, and new distinct event in `../zunera-frontend/src/components/notifications/__tests__/NotificationPreferences.spec.js` and `../zunera-frontend/e2e/notifications.spec.js`.

**Checkpoint**: Story-specific UI and journey tests pass against the established backend contract.

---

## Phase 16: Final Frontend and End-to-End Verification

**Purpose**: Verify accessibility, browser performance, user understanding, and complete release evidence.

- [X] T069 Add frontend accessibility/locale/theme/320px/200%-zoom regression coverage and no background announcement flood in `../zunera-frontend/e2e/notifications.spec.js` and `../zunera-frontend/src/components/notifications/__tests__/NotificationAccessibility.spec.js`.
- [X] T070 Measure 10,000-item first-page/filter response and browser usable-content time across at least 20 attempts; document 95th-percentile results in `specs/017-notifications/checklists/timeliness-performance.md`.
- [ ] T071 Conduct the uncoached 10-participant find/action-state usability checks for SC-001 and SC-007; record anonymized pass rates in `specs/017-notifications/checklists/usability.md`.
- [X] T072 Run full backend tests, Pint, frontend unit/lint/build, isolated notification Playwright, and affected card/budget/recurrence/goal regressions; record outcomes in `specs/017-notifications/checklists/release-verification.md`.
- [X] T073 Reconcile actual API responses, source-mutation audit, scheduler runtime, and implemented states against `specs/017-notifications/contracts/notifications-api.yaml`, `specs/017-notifications/spec.md`, and `specs/017-notifications/quickstart.md`; update only confirmed design drift.

**Checkpoint**: All seven success criteria have evidence or an explicit unmet result.

---

## Dependencies & Execution Order

### Feature-wide gate

- Setup T001–T003 → Foundation T004–T018 → all backend stories T019–T045 → backend verification T046–T049 → all frontend stories T050–T068 → final verification T069–T073.
- T049 is the mandatory feature-wide backend gate. No frontend test or implementation task starts before it passes. Backend contracts, authorization, validation, services, source integration, and automated tests for all six stories must be complete.
- Each story remains independently testable through its backend acceptance fixtures and later UI journey, though frontend work waits for the complete backend gate.

### Story dependency graph

```text
Setup T001–T003 → Foundation T004–T018
  → Backend US1 T019–T024 → Backend US2 T025–T029 → Backend US3 T030–T033
  → Backend US4 T034–T038 → Backend US5 T039–T042 → Backend US6 T043–T045
  → Feature-wide backend gate T046–T049
  → Frontend US1 T050–T056 → Frontend US2 T057–T058 → Frontend US3 T059–T060
  → Frontend US4 T061–T062 → Frontend US5 T063–T064 → Frontend US6 T065–T068
  → Final verification T069–T073
```

### Parallel execution examples

- Backend: T019 and T020 cover distinct US1 test files; T025 statement tests and T030 recurrence tests can be drafted independently after foundation. T034 budget and T039 goal tests are independent. Coordinate source-service edits where stories touch the same files.
- Backend gate: T046 concurrency/scale and T047 privacy/retention tests use separate files. Complete T048 measurements and T049 full gate before any frontend work.
- Frontend after T049: T050 service/store tests and isolated component design can proceed in separate files; US2–US6 destination work may proceed when the US1 shared center exists, without parallel edits to `notificationDestination.js` or `notifications.spec.js`.

## Implementation Strategy

1. Finish all backend stories and the T049 feature-wide gate, honoring the constitution’s backend-before-frontend rule.
2. **MVP**: Add frontend US1 to the complete backend. Demonstrate the center with seeded and source-derived events, owner-safe counts, pages, read controls, and navigation.
3. Add frontend US2–US6 in priority order, testing each user journey against the verified API.
4. Finish T069–T073 accessibility, browser performance, usability, and release evidence. Stop optional testing once identified risks are adequately verified.

## Notes

- `[P]` means work on separate files without an incomplete task dependency; it never overrides T049’s frontend gate.
- T002 supplies the accepted-mutation inventory and may identify extra source-service hook files beyond those named in story tasks.
- Source financial calculation changes are outside scope; notification hooks observe accepted state only.
