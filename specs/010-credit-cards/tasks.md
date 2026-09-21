# Tasks: Credit Cards

**Input**: Design documents from `specs/010-credit-cards/`
**Paths**: All paths are relative to the `zunera-specs` repository root; `../zunera-backend` and `../zunera-frontend` are sibling repositories.
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/credit-cards-api.yaml`, `quickstart.md`

**Tests**: Automated backend, frontend, contract, and end-to-end tests are required by the project constitution and plan.

**Organization**: Tasks are grouped by user story so each story remains independently testable and deliverable. Within every story, backend work completes and is verified before dependent frontend work begins.

## Phase 1: Setup and contract baseline

**Purpose**: Confirm shared feature context and establish a testable API contract.

- [X] T001 Verify the `010-credit-cards` branch is checked out in `zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.
- [X] T002 [P] Validate `specs/010-credit-cards/contracts/credit-cards-api.yaml` with Redocly.
- [X] T003 [P] Add Credit Cards API fixture and authentication helpers in `../zunera-backend/tests/Support/CreditCards/` following existing Financial Accounts test conventions.
- [ ] T004 [P] Add representative request/response contract fixtures in `specs/010-credit-cards/contracts/fixtures/credit-cards.yaml` for card, purchase, statement, payment, and credit-event examples.

---

## Phase 2: Foundational financial domain

**Purpose**: Create authoritative card-domain primitives used by every user story. No user story begins until this phase is complete.

- [X] T005 Create credit-card persistence migrations in `../zunera-backend/database/migrations/` for cards, purchases, installments, statements, statement payments, credit events, credit applications, and mutation requests from `specs/010-credit-cards/data-model.md`.
- [X] T006 [P] Create Eloquent models, relationships, casts, factories, and ownership scopes in `../zunera-backend/app/Models/CreditCard.php`, `../zunera-backend/app/Models/CreditCardPurchase.php`, `../zunera-backend/app/Models/CreditCardInstallment.php`, `../zunera-backend/app/Models/CreditCardStatement.php`, `../zunera-backend/app/Models/CreditCardStatementPayment.php`, `../zunera-backend/app/Models/CreditCardCreditEvent.php`, `../zunera-backend/app/Models/CreditCardCreditApplication.php`, and `../zunera-backend/app/Models/CreditCardMutationRequest.php`.
- [X] T007 [P] Create card, purchase, installment, statement, payment, and credit-event enums plus immutable DTOs in `../zunera-backend/app/Enums/CreditCards/` and `../zunera-backend/app/Data/CreditCards/`.
- [X] T008 [P] Add 100 representative statement-cycle unit cases for currency allocation, day clamping, inclusive closing-day allocation and next-calendar-date finalization, due-date calculation, leap-year, year-boundary, and 1/360-installment boundaries in `../zunera-backend/tests/Unit/CreditCards/BillingCycleCalculatorTest.php` and `../zunera-backend/tests/Unit/CreditCards/InstallmentAllocatorTest.php`.
- [X] T009 Implement exact-cent installment allocation and America/Sao_Paulo billing-cycle calculation in `../zunera-backend/app/Services/CreditCards/InstallmentAllocator.php` and `../zunera-backend/app/Services/CreditCards/BillingCycleCalculator.php`.
- [X] T010 Implement mutation idempotency storage, request fingerprinting, same-request replay, fresh-key explicit over-limit confirmation, and lock ordering in `../zunera-backend/app/Services/CreditCards/CreditCardMutationIdempotencyService.php` and `../zunera-backend/app/Exceptions/CreditCards/`.
- [X] T011 Implement card balance, available-credit, statement-total, and oldest-unpaid-credit-application projection in `../zunera-backend/app/Services/CreditCards/CreditCardObligationReconciler.php`.
- [X] T012 Add ownership authorization and error rendering in `../zunera-backend/app/Policies/CreditCardPolicy.php`, register that policy for `CreditCard` in `../zunera-backend/app/Providers/AppServiceProvider.php`, and map typed Credit Cards exceptions in `../zunera-backend/bootstrap/app.php` (Laravel 13 slim has no `AuthServiceProvider` or `app/Exceptions/Handler.php`).
- [X] T013 Add shared API resource representations and financial-mutation idempotency header handling in `../zunera-backend/app/Http/Resources/CreditCards/` and `../zunera-backend/app/Http/Middleware/`.
- [ ] T014 Verify persistence, allocation, idempotency, ownership, and reconciliation in `../zunera-backend/tests/Unit/CreditCards/` and `../zunera-backend/tests/Feature/CreditCards/FoundationalCreditCardDomainTest.php`.

**Checkpoint**: The shared domain represents a card, calculates billing periods deterministically, safely replays mutations, and reconciles obligations without touching ordinary financial-account balances.

---

## Phase 3: User Story 1 — Register and manage credit cards (Priority: P1) 🎯 MVP

**Goal**: Users can create, view, edit, and archive their own cards while preserving history and preventing new activity on archived cards.

**Independent Test**: Create a card, view its detail, change a later-applicable billing configuration, archive it only after its debt and credit reach zero, and confirm another user cannot read or mutate it.

### Backend for User Story 1

- [X] T015 [P] [US1] Write card creation, listing, detail, update, ownership isolation, archive-eligibility, and billing-day-immutability coverage (after a closing/due day update, existing statements keep their original period, closing date, due date, and installment assignments while later purchases use the new billing days) in `../zunera-backend/tests/Feature/CreditCards/CreditCardManagementTest.php`.
- [X] T016 [P] [US1] Write validation coverage for positive limits, valid days, safe identifiers, status, and no sensitive credentials in `../zunera-backend/tests/Feature/CreditCards/CreditCardManagementValidationTest.php`.
- [X] T017 [US1] Implement create, list, detail, update, and archive lifecycle rules in `../zunera-backend/app/Services/CreditCards/CreditCardService.php`.
- [X] T018 [US1] Implement create/update validation in `../zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardRequest.php` and `../zunera-backend/app/Http/Requests/CreditCards/UpdateCreditCardRequest.php`.
- [X] T019 [US1] Implement summary/detail resources, including limit, used credit, available credit, status, and historical indicators, in `../zunera-backend/app/Http/Resources/CreditCards/CreditCardResource.php` and `../zunera-backend/app/Http/Resources/CreditCards/CreditCardDetailResource.php`.
- [X] T020 [US1] Add authenticated card controller actions and routes in `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardController.php` and `../zunera-backend/routes/api.php`.
- [ ] T021 [US1] Align create, read, update, and archive examples in `specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [X] T022 [US1] Run backend feature and contract checks from `../zunera-backend/tests/Feature/CreditCards/CreditCardManagementTest.php` and `specs/010-credit-cards/contracts/credit-cards-api.yaml`.

### Frontend for User Story 1

- [X] T023 [P] [US1] Add formatter tests for BRL amounts, calendar dates, installment sequence labels, statement status, over-limit, and available-credit display in `../zunera-frontend/src/utils/credit-cards/__tests__/creditCardFormatters.spec.js`.
- [X] T024 [US1] Implement pure presentation formatters that only render server-provided values, with no client-side money, status, or cycle derivation, in `../zunera-frontend/src/utils/credit-cards/creditCardFormatters.js`.
- [X] T025 [P] [US1] Add card list, detail, create, update, archive, stable same-mutation `Idempotency-Key` generation, and safe replay-retry API-service tests in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js`.
- [X] T026 [P] [US1] Add loading, feedback, and archived-card state tests in `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [X] T027 [US1] Implement card API access, stable same-mutation `Idempotency-Key` reuse for network retry/replay, fresh-key user-confirmed mutation submission, and state orchestration in `../zunera-frontend/src/services/creditCardService.js` and `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [X] T028 [US1] Implement accessible responsive card list, detail, and create/edit/archive flows in `../zunera-frontend/src/views/credit-cards/CreditCardsListView.vue`, `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`, and `../zunera-frontend/src/components/credit-cards/CreditCardForm.vue`.
- [ ] T029 [US1] Register navigation, routes, localized feedback, and management browser coverage, including the spec-local fixture builders used by the Credit Cards browser suite, in `../zunera-frontend/src/router/index.js`, `../zunera-frontend/src/layouts/AppShell.vue`, `../zunera-frontend/src/i18n/messages.js`, and `../zunera-frontend/e2e/credit-cards.spec.js`. (Routes, navigation, and i18n are done; the Playwright management journey is still open.)

**Checkpoint**: A user manages only their own non-sensitive cards; archives stay readable and correctly guarded by financial state.

---

## Phase 4: User Story 2 — Record purchases and installments (Priority: P1) 🎯 MVP

**Goal**: Users record one-time and interest-free installment purchases, understand their assigned statement, and see accurate available credit without directly changing a financial-account balance.

**Independent Test**: Record purchases on and around closing day, including a non-even installment amount; verify allocation sums exactly, installment statements are predictable, categories are owned expense categories, and account balances remain unchanged.

### Backend for User Story 2

- [X] T030 [P] [US2] Write one-time purchase, closing-day assignment, archived-card rejection, typed over-limit confirmation-required flow with a fresh confirmation key and same-request network replay, exact negative availability, and no-account-balance-movement coverage in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseTest.php`.
- [X] T031 [P] [US2] Write exact installment rounding, sequential statement allocation, category ownership, and purchase idempotency coverage in `../zunera-backend/tests/Feature/CreditCards/CreditCardInstallmentPurchaseTest.php`.
- [X] T032 [US2] Implement purchase creation, category authorization, projected-limit checks, explicit over-limit confirmation gating, and immutable installment generation in `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php`.
- [X] T033 [US2] Implement positive-money, date, owned-card/category, interest-free installment-count, and explicit over-limit-confirmation validation in `../zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardPurchaseRequest.php`.
- [ ] T034 [US2] Implement purchase/installment resources with statement period, due date, realization state, and sequence in `../zunera-backend/app/Http/Resources/CreditCards/CreditCardPurchaseResource.php` and `../zunera-backend/app/Http/Resources/CreditCards/CreditCardInstallmentResource.php`.
- [X] T035 [US2] Add authorized purchase creation/listing controller actions and routes in `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardPurchaseController.php` and `../zunera-backend/routes/api.php`.
- [ ] T036 [US2] Align purchase operations, idempotency requirements, typed over-limit confirmation-required/retry responses, and error examples in `specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T037 [US2] Run purchase/allocator suites in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseTest.php`, `../zunera-backend/tests/Feature/CreditCards/CreditCardInstallmentPurchaseTest.php`, and `../zunera-backend/tests/Unit/CreditCards/InstallmentAllocatorTest.php`.

### Frontend for User Story 2

- [X] T038 [P] [US2] Add purchase submission, fresh-key over-limit confirmation, same-request retry with stable idempotency key, installment preview, category failure, and assigned-statement client tests in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js` and `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [X] T039 [P] [US2] Add currency input, installment preview, closing-date explanation, typed over-limit warning/confirmation with negative availability, and accessible-error tests in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardPurchaseForm.spec.js`.
- [ ] T040 [US2] Extend client state for purchases, installments, statement assignment previews, and idempotent feedback in `../zunera-frontend/src/services/creditCardService.js` and `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [X] T041 [US2] Implement purchase entry, over-limit warning/explicit confirmation with negative availability, and installment schedule presentation in `../zunera-frontend/src/components/credit-cards/CreditCardPurchaseForm.vue`, `../zunera-frontend/src/components/credit-cards/InstallmentSchedule.vue`, and `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [X] T042 [US2] Add browser coverage for single/installment purchases, closing-day messaging, over-limit confirmation with a fresh request key, confirmed-request network retry, and unchanged paying-account balance in `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: Card spending is recorded once, installments reconcile exactly, and ordinary financial accounts are not charged.

---

## Phase 5: User Story 3 — Review statements and pay them (Priority: P1) 🎯 MVP

**Goal**: Users inspect open and historical statements, make partial or full payments from owned financial accounts, and see accurate obligations and payment history.

**Independent Test**: Close a statement with purchases, pay it in two idempotent payments from an owned account, verify account balance moves once per effective payment, and verify partial, paid, overdue, and invalid-payment states.

### Backend for User Story 3

- [X] T043 [P] [US3] Write inclusive-closing-day/next-calendar-date finalization plus open, closed, partially paid, paid, overdue, and zero-amount statement coverage in `../zunera-backend/tests/Feature/CreditCards/CreditCardStatementTest.php`.
- [X] T044 [P] [US3] Write 100 representative partial/full settlement, account ownership, intentional overdraft, replay, edit/remove/restore, and payment-history lifecycle cases in `../zunera-backend/tests/Feature/CreditCards/CreditCardStatementPaymentTest.php`.
- [X] T045 [US3] Implement statement refresh, close finalization, state transitions, history, and retrieval in `../zunera-backend/app/Services/CreditCards/CreditCardStatementService.php`.
- [X] T046 [US3] Implement payment creation, correction, removal, restoration, account-balance deltas, and double-application prevention in `../zunera-backend/app/Services/CreditCards/CreditCardStatementPaymentService.php`.
- [X] T047 [US3] Reuse financial-account balance conventions through a card-payment reconciliation adapter in `../zunera-backend/app/Services/CreditCards/CreditCardPaymentAccountReconciler.php` and `../zunera-backend/app/Services/Transactions/TransactionBalanceReconciler.php`.
- [X] T048 [US3] Implement payment, correction, removal, and restoration validation in `../zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardStatementPaymentRequest.php` and `../zunera-backend/app/Http/Requests/CreditCards/UpdateCreditCardStatementPaymentRequest.php`.
- [ ] T049 [US3] Implement statement, line-item, and payment-history resources in `../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementResource.php`, `../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementDetailResource.php`, and `../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementPaymentResource.php`.
- [X] T050 [US3] Add statement/payment controller actions and routes in `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardStatementController.php`, `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardStatementPaymentController.php`, and `../zunera-backend/routes/api.php`.
- [ ] T051 [US3] Align statement state, payment, correction, and replay responses in `specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T052 [US3] Run statement/payment backend regression suites in `../zunera-backend/tests/Feature/CreditCards/CreditCardStatementTest.php` and `../zunera-backend/tests/Feature/CreditCards/CreditCardStatementPaymentTest.php`.

### Frontend for User Story 3

- [ ] T053 [P] [US3] Add statement query, partial-payment same-request retry with stable idempotency key, state, and payment-history reload tests in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js` and `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [ ] T054 [P] [US3] Add due-date/status, line-item, partial-total, and account-validation component tests in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardStatementPanel.spec.js` and `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardStatementPaymentDialog.spec.js`.
- [X] T055 [US3] Extend client state for statements, details, history, and settlement mutations in `../zunera-frontend/src/services/creditCardService.js` and `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [X] T056 [US3] Implement statement list/detail, payment history, and partial-payment flows in `../zunera-frontend/src/components/credit-cards/CreditCardStatementList.vue`, `../zunera-frontend/src/components/credit-cards/CreditCardStatementPanel.vue`, `../zunera-frontend/src/components/credit-cards/CreditCardStatementPaymentDialog.vue`, and `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [X] T057 [P] [US3] After T056, add effective-payment edit, removal, restoration, and paying-account reassignment client coverage for restated statement/account values, preserved payment history, and archived-account rejection in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js`, `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`, and `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardStatementPaymentDialog.spec.js`.
- [X] T058 [US3] After T057, implement effective-payment edit, removal, restoration, and paying-account reassignment flows with refreshed statement outstanding, account balance, payment history, and impact state in `../zunera-frontend/src/components/credit-cards/CreditCardStatementPaymentDialog.vue`, `../zunera-frontend/src/components/credit-cards/CreditCardStatementPanel.vue`, `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`, and `../zunera-frontend/src/services/creditCardService.js`.
- [X] T059 [US3] Add end-to-end coverage for partial/full payment, payment edit/removal/restoration with paying-account reassignment, duplicate-submit prevention, account movement, and visible state changes in `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: Payments settle the obligation and affect only the selected financial account, without creating a second expense.

---

## Phase 6: User Story 4 — Understand cards in budgets, dashboard, and history (Priority: P2)

**Goal**: Users understand card obligations and spending alongside their financial overview without double counting purchase and payment events.

**Independent Test**: Create an open-statement installment and verify it is Expected in its budget month; close it and verify one Realized expense appears in budget, dashboard, and history, while payment adds no category expense.

### Backend for User Story 4

- [X] T060 [P] [US4] Write 100 representative purchase-and-payment pairs covering pending installments, close realization, category/month allocation, and payment exclusion in `../zunera-backend/tests/Feature/CreditCards/CreditCardBudgetIntegrationTest.php`.
- [X] T061 [P] [US4] Write card-obligation, available-credit, upcoming-due, history, and no-double-count coverage in `../zunera-backend/tests/Feature/CreditCards/CreditCardDashboardAndHistoryIntegrationTest.php`.
- [X] T062 [US4] Integrate expected versus realized installment recognition into `../zunera-backend/app/Services/Budgets/BudgetCalculationService.php` and `../zunera-backend/app/Services/CreditCards/CreditCardBudgetProjectionService.php`.
- [X] T063 [US4] Integrate card obligations, available credit, and upcoming dues into `../zunera-backend/app/Services/FinancialDashboard/DashboardSummaryService.php`, `../zunera-backend/app/Services/FinancialDashboard/DashboardUpcomingActivityService.php`, and `../zunera-backend/app/Services/CreditCards/CreditCardDashboardProjectionService.php`.
- [ ] T064 [US4] Add recognized card expenses as discriminated history entries and exclude payments from category totals in `../zunera-backend/app/Services/FinancialHistory/FinancialHistoryService.php` and `../zunera-backend/app/Http/Resources/FinancialHistory/FinancialHistoryResource.php`.
- [ ] T065 [US4] Add dashboard card projection endpoint and route in `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardDashboardController.php` and `../zunera-backend/routes/api.php`.
- [ ] T066 [US4] Align dashboard summaries and `credit_card_expense` history discriminator in `specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T067 [US4] Run integration suites in `../zunera-backend/tests/Feature/CreditCards/CreditCardBudgetIntegrationTest.php` and `../zunera-backend/tests/Feature/CreditCards/CreditCardDashboardAndHistoryIntegrationTest.php`.

### Frontend for User Story 4

- [ ] T068 [P] [US4] Add dashboard card-summary service/store tests in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js` and `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [X] T069 [P] [US4] Add obligation/available-credit, upcoming-due, and non-color-only status tests in `../zunera-frontend/src/components/dashboard/__tests__/CreditCardSummary.spec.js`.
- [X] T070 [US4] Implement card summary loading and presentation in `../zunera-frontend/src/services/creditCardService.js`, `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`, `../zunera-frontend/src/components/dashboard/CreditCardSummary.vue`, and `../zunera-frontend/src/views/dashboard/FinancialDashboardView.vue`.
- [X] T071 [US4] Update expected/realized card spending and history presentation, and add the recurring-card-unsupported notice with manual-card-purchase guidance, in `../zunera-frontend/src/views/budgets/BudgetsView.vue`, `../zunera-frontend/src/views/transactions/TransactionsListView.vue` (existing `listFinancialHistory` screen), and `../zunera-frontend/src/views/recurring-transactions/RecurringTransactionsListView.vue`.
- [X] T072 [US4] Add browser coverage for budget recognition, dashboard obligations, history entries, payment non-duplication, and the recurring-card-unsupported notice in `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: Account balances remain separate from card liabilities; a card expense is recognized exactly once even when later paid.

---

## Phase 7: User Story 5 — Correct purchases and record refunds (Priority: P3)

**Goal**: Users make traceable corrections, cancellations, and refunds while installments, statements, credits, obligations, and available credit reconcile correctly.

**Independent Test**: Correct an open purchase and refund a closed-statement purchase; verify traceable events, updated obligation, oldest-unpaid automatic credit application, and archive cannot hide debt or residual credit.

### Backend for User Story 5

- [X] T073 [P] [US5] Write pre-close card/category/amount/date/installment-count correction and cancellation coverage in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseCorrectionTest.php`.
- [X] T074 [P] [US5] Write full/partial refund, post-closing correction credit-event, oldest-unpaid application, available-credit, and archive coverage in `../zunera-backend/tests/Feature/CreditCards/CreditCardRefundAndCreditEventTest.php`.
- [X] T075 [US5] Implement allowed open-purchase corrections, reallocation, immutable history, and closed-purchase rejection in `../zunera-backend/app/Services/CreditCards/CreditCardPurchaseCorrectionService.php`.
- [X] T076 [US5] Implement refunds, cancellations, post-closing correction credit events, oldest-unpaid auto-application, and card-credit reconciliation in `../zunera-backend/app/Services/CreditCards/CreditCardCreditEventService.php` and `../zunera-backend/app/Services/CreditCards/CreditCardObligationReconciler.php`.
- [X] T077 [US5] Implement correction/credit-event validation in `../zunera-backend/app/Http/Requests/CreditCards/UpdateCreditCardPurchaseRequest.php` and `../zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardCreditEventRequest.php`.
- [ ] T078 [US5] Implement correction, credit-event, and application-history resources in `../zunera-backend/app/Http/Resources/CreditCards/CreditCardCreditEventResource.php` and `../zunera-backend/app/Http/Resources/CreditCards/CreditCardCreditApplicationResource.php`.
- [ ] T079 [US5] Add correction/credit-event controller actions and routes in `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardPurchaseController.php`, `../zunera-backend/app/Http/Controllers/Api/V1/CreditCardCreditEventController.php`, and `../zunera-backend/routes/api.php`.
- [ ] T080 [US5] Align correction, refund, cancellation, credit-application, and conflict responses in `specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T081 [US5] Run correction/refund/reconciliation suites in `../zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseCorrectionTest.php`, `../zunera-backend/tests/Feature/CreditCards/CreditCardRefundAndCreditEventTest.php`, and `../zunera-backend/tests/Unit/CreditCards/CreditCardObligationReconcilerTest.php`.

### Frontend for User Story 5

- [ ] T082 [P] [US5] Add correction, refund, cancellation, application-history, and archive-conflict tests in `../zunera-frontend/src/services/__tests__/creditCardService.spec.js` and `../zunera-frontend/src/stores/credit-cards/__tests__/creditCardStore.spec.js`.
- [ ] T083 [P] [US5] Add pre-close correction, post-closing correction-credit-event, refund, card-credit explanation, and invalid-state tests in `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardCorrectionDialog.spec.js` and `../zunera-frontend/src/components/credit-cards/__tests__/CreditCardCreditEventDialog.spec.js`.
- [ ] T084 [US5] Extend client state for correction, refund, cancellation, credit-event, and application-history mutations in `../zunera-frontend/src/services/creditCardService.js` and `../zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T085 [US5] Implement pre-close correction, post-closing correction-credit-event, refund/cancellation, and card-credit history flows in `../zunera-frontend/src/components/credit-cards/CreditCardCorrectionDialog.vue`, `../zunera-frontend/src/components/credit-cards/CreditCardCreditEventDialog.vue`, `../zunera-frontend/src/components/credit-cards/CreditCardCreditHistory.vue`, and `../zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [ ] T086 [US5] Add browser coverage for correction, refund, auto-credit application, and archive blocking in `../zunera-frontend/e2e/credit-cards.spec.js`.

**Checkpoint**: Corrections/refunds stay auditable, repair the card position, and never silently delete history or duplicate spending.

---

## Phase 8: Polish and release readiness

**Purpose**: Confirm quality, performance, accessibility, documentation, and contract consistency across the feature.

- [ ] T087 [P] Add cross-domain authorization regression coverage in `../zunera-backend/tests/Feature/CreditCards/CreditCardAuthorizationTest.php`.
- [ ] T088 [P] Seed 10,000 combined owned financial/card movements; assert card list, statement read, and affected projections each complete within 2 seconds; and capture query-plan evidence before adding indexes in `../zunera-backend/tests/Feature/CreditCards/CreditCardQueryPerformanceTest.php` and `../zunera-backend/tests/Performance/CreditCards/query-plans.md`.
- [ ] T089 [P] Add responsive theme and keyboard/screen-reader browser checks in `../zunera-frontend/e2e/credit-cards.spec.js`.
- [ ] T090 Validate lifecycle examples and manual acceptance checks in `specs/010-credit-cards/quickstart.md`.
- [ ] T091 Run full backend quality and Credit Cards suites against the running container with `docker exec zunera-backend-app-1 php artisan test` plus `docker exec zunera-backend-app-1 vendor/bin/pint --format=agent` (start the stack first with `../zunera-backend/start.sh` if it is not running).
- [ ] T092 Run frontend unit and Playwright suites from `../zunera-frontend/` with `npm run test:unit -- --run`, `npm run build`, and `CI=1 npm run test:e2e -- e2e/credit-cards.spec.js`.
- [ ] T093 Validate final contract at `specs/010-credit-cards/contracts/credit-cards-api.yaml` with Redocly.
- [ ] T094 Run frontend lint and production-build verification with `npm run lint` and `npm run build` from `../zunera-frontend/`.

---

## Dependencies and execution order

### Phase dependencies

- **Setup (Phase 1)**: No dependencies.
- **Foundational domain (Phase 2)**: Depends on Phase 1; blocks all stories.
- **US1 (Phase 3)**: Depends on Phase 2 and provides card lifecycle.
- **US2 (Phase 4)**: Depends on Phase 2 and active cards from US1.
- **US3 (Phase 5)**: Depends on Phase 2, US1 cards, and US2 purchases.
- **US4 (Phase 6)**: Depends on US2 recognition data and US3 settlement behavior.
- **US5 (Phase 7)**: Depends on US2 purchase lifecycle and US3 statement lifecycle.
- **Polish (Phase 8)**: Depends on all intended stories.

### User story dependency graph

```text
Setup → Foundational → US1 → US2 → US3 → US4
                              └────→ US5
```

### Parallel opportunities

- T002–T004 may run in parallel after T001.
- T006–T008 may run in parallel after T005; T010 and T012–T013 follow their prerequisites.
- Each story begins with its marked `[P]` backend tests. After the backend checkpoint, its marked `[P]` frontend tests may run in parallel.
- Frontend unit specs live beside their source under `src/**/__tests__/`; the
  whole Credit Cards browser suite lives in `e2e/credit-cards.spec.js`.
- US4 and US5 may run in parallel after US3 if shared reconciliation and contract edits are coordinated.
- T085–T087 may run in parallel before final validation.

## Parallel example: User Story 2

```bash
Task: "T030 [US2] CreditCardPurchaseTest.php"
Task: "T031 [US2] CreditCardInstallmentPurchaseTest.php"

Task: "T038 [US2] client/store tests"
Task: "T039 [US2] purchase-form component test"
```

## Parallel example: User Story 3

```bash
Task: "T043 [US3] CreditCardStatementTest.php"
Task: "T044 [US3] CreditCardStatementPaymentTest.php"

Task: "T053 [US3] client/store tests"
Task: "T054 [US3] statement component tests"
Task: "T057 [US3] payment edit/removal/restoration client tests"
```

## Implementation strategy

### MVP first (US1 + US2 + US3)

1. Complete Phases 1–2 for authoritative money, ownership, locking, and idempotency rules.
2. Deliver US1, then US2, then US3; validate the P1 flow end to end.
3. Add US4 for visibility and US5 for traceable remediation without changing core recognition.
4. Complete Phase 8 before release; do not defer authorization, idempotency, accessibility, or financial-consistency checks.

### Suggested MVP scope

Phases 1–5 provide the smallest independent release: card management, purchases/installments, statements, and partial/full statement payments. US4 and US5 extend visibility and remediation without changing the established core recognition model.
