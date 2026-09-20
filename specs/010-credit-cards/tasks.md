# Tasks: Credit Cards

**Input**: Design documents from `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/credit-cards-api.yaml`, `quickstart.md`

**Tests**: Automated backend, frontend, contract, and end-to-end tests are required by the project constitution and plan.

**Organization**: Tasks are grouped by user story so each story remains independently testable and deliverable. Within every story, backend work completes and is verified before dependent frontend work begins.

## Phase 1: Setup and contract baseline

**Purpose**: Confirm shared feature context and establish a testable API contract.

- [ ] T001 Verify the `010-credit-cards` branch is checked out in `/home/ederson/workspace/zunera/zunera-specs`, `/home/ederson/workspace/zunera/zunera-backend`, and `/home/ederson/workspace/zunera/zunera-frontend`.
- [ ] T002 [P] Validate `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml` with Redocly.
- [ ] T003 [P] Add Credit Cards API fixture and authentication helpers in `/home/ederson/workspace/zunera/zunera-backend/tests/Support/CreditCards/` following existing Financial Accounts test conventions.
- [ ] T004 [P] Add representative request/response contract fixtures in `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/fixtures/credit-cards.yaml` for card, purchase, statement, payment, and credit-event examples.

---

## Phase 2: Foundational financial domain

**Purpose**: Create authoritative card-domain primitives used by every user story. No user story begins until this phase is complete.

- [ ] T005 Create credit-card persistence migrations in `/home/ederson/workspace/zunera/zunera-backend/database/migrations/` for cards, purchases, installments, statements, statement payments, credit events, credit applications, and mutation requests from `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/data-model.md`.
- [ ] T006 [P] Create Eloquent models, relationships, casts, factories, and ownership scopes in `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCard.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCardPurchase.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCardInstallment.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCardStatement.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCardStatementPayment.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCardCreditEvent.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCardCreditApplication.php`, and `/home/ederson/workspace/zunera/zunera-backend/app/Models/CreditCardMutationRequest.php`.
- [ ] T007 [P] Create card, purchase, installment, statement, payment, and credit-event enums plus immutable DTOs in `/home/ederson/workspace/zunera/zunera-backend/app/Enums/CreditCards/` and `/home/ederson/workspace/zunera/zunera-backend/app/Data/CreditCards/`.
- [ ] T008 [P] Add 100 representative statement-cycle unit cases for currency allocation, day clamping, inclusive closing-day allocation and next-calendar-date finalization, due-date calculation, leap-year, year-boundary, and 1/360-installment boundaries in `/home/ederson/workspace/zunera/zunera-backend/tests/Unit/CreditCards/BillingCycleCalculatorTest.php` and `/home/ederson/workspace/zunera/zunera-backend/tests/Unit/CreditCards/InstallmentAllocatorTest.php`.
- [ ] T009 Implement exact-cent installment allocation and America/Sao_Paulo billing-cycle calculation in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/InstallmentAllocator.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/BillingCycleCalculator.php`.
- [ ] T010 Implement mutation idempotency storage, request fingerprinting, same-request replay, fresh-key explicit over-limit confirmation, and lock ordering in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardMutationIdempotencyService.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Exceptions/CreditCards/`.
- [ ] T011 Implement card balance, available-credit, statement-total, and oldest-unpaid-credit-application projection in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardObligationReconciler.php`.
- [ ] T012 Add ownership authorization and error rendering in `/home/ederson/workspace/zunera/zunera-backend/app/Policies/CreditCardPolicy.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Providers/AuthServiceProvider.php`, and `/home/ederson/workspace/zunera/zunera-backend/app/Exceptions/Handler.php`.
- [ ] T013 Add shared API resource representations and financial-mutation idempotency header handling in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Middleware/`.
- [ ] T014 Verify persistence, allocation, idempotency, ownership, and reconciliation in `/home/ederson/workspace/zunera/zunera-backend/tests/Unit/CreditCards/` and `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/FoundationalCreditCardDomainTest.php`.

**Checkpoint**: The shared domain represents a card, calculates billing periods deterministically, safely replays mutations, and reconciles obligations without touching ordinary financial-account balances.

---

## Phase 3: User Story 1 — Register and manage credit cards (Priority: P1) 🎯 MVP

**Goal**: Users can create, view, edit, and archive their own cards while preserving history and preventing new activity on archived cards.

**Independent Test**: Create a card, view its detail, change a later-applicable billing configuration, archive it only after its debt and credit reach zero, and confirm another user cannot read or mutate it.

### Backend for User Story 1

- [ ] T015 [P] [US1] Write card creation, listing, detail, update, ownership isolation, and archive-eligibility coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardManagementTest.php`.
- [ ] T016 [P] [US1] Write validation coverage for positive limits, valid days, safe identifiers, status, and no sensitive credentials in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardManagementValidationTest.php`.
- [ ] T017 [US1] Implement create, list, detail, update, and archive lifecycle rules in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardService.php`.
- [ ] T018 [US1] Implement create/update validation in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardRequest.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Requests/CreditCards/UpdateCreditCardRequest.php`.
- [ ] T019 [US1] Implement summary/detail resources, including limit, used credit, available credit, status, and historical indicators, in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardResource.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardDetailResource.php`.
- [ ] T020 [US1] Add authenticated card controller actions and routes in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Controllers/Api/V1/CreditCardController.php` and `/home/ederson/workspace/zunera/zunera-backend/routes/api.php`.
- [ ] T021 [US1] Align create, read, update, and archive examples in `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T022 [US1] Run backend feature and contract checks from `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardManagementTest.php` and `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml`.

### Frontend for User Story 1

- [ ] T023 [P] [US1] Add card list, detail, create, update, archive, stable same-mutation `Idempotency-Key` generation, and safe replay-retry API-service tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/services/creditCardService.spec.js`.
- [ ] T024 [P] [US1] Add loading, feedback, and archived-card state tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/stores/creditCardStore.spec.js`.
- [ ] T025 [US1] Implement card API access, stable same-mutation `Idempotency-Key` reuse for network retry/replay, fresh-key user-confirmed mutation submission, and state orchestration in `/home/ederson/workspace/zunera/zunera-frontend/src/services/creditCardService.js` and `/home/ederson/workspace/zunera/zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T026 [US1] Implement accessible responsive card list, detail, and create/edit/archive flows in `/home/ederson/workspace/zunera/zunera-frontend/src/views/credit-cards/CreditCardsView.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`, and `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardForm.vue`.
- [ ] T027 [US1] Register navigation, routes, localized feedback, and management browser coverage, including fixture builders, in `/home/ederson/workspace/zunera/zunera-frontend/src/router/index.js`, `/home/ederson/workspace/zunera/zunera-frontend/src/layouts/AppShell.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/i18n/messages.js`, `/home/ederson/workspace/zunera/zunera-frontend/tests/e2e/fixtures/creditCards.js`, and `/home/ederson/workspace/zunera/zunera-frontend/tests/e2e/credit-cards/credit-card-management.spec.js`.

**Checkpoint**: A user manages only their own non-sensitive cards; archives stay readable and correctly guarded by financial state.

---

## Phase 4: User Story 2 — Record purchases and installments (Priority: P1) 🎯 MVP

**Goal**: Users record one-time and interest-free installment purchases, understand their assigned statement, and see accurate available credit without directly changing a financial-account balance.

**Independent Test**: Record purchases on and around closing day, including a non-even installment amount; verify allocation sums exactly, installment statements are predictable, categories are owned expense categories, and account balances remain unchanged.

### Backend for User Story 2

- [ ] T028 [P] [US2] Write one-time purchase, closing-day assignment, archived-card rejection, typed over-limit confirmation-required flow with a fresh confirmation key and same-request network replay, exact negative availability, and no-account-balance-movement coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseTest.php`.
- [ ] T029 [P] [US2] Write exact installment rounding, sequential statement allocation, category ownership, and purchase idempotency coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardInstallmentPurchaseTest.php`.
- [ ] T030 [US2] Implement purchase creation, category authorization, projected-limit checks, explicit over-limit confirmation gating, and immutable installment generation in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardPurchaseService.php`.
- [ ] T031 [US2] Implement positive-money, date, owned-card/category, interest-free installment-count, and explicit over-limit-confirmation validation in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardPurchaseRequest.php`.
- [ ] T032 [US2] Implement purchase/installment resources with statement period, due date, realization state, and sequence in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardPurchaseResource.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardInstallmentResource.php`.
- [ ] T033 [US2] Add authorized purchase creation/listing controller actions and routes in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Controllers/Api/V1/CreditCardPurchaseController.php` and `/home/ederson/workspace/zunera/zunera-backend/routes/api.php`.
- [ ] T034 [US2] Align purchase operations, idempotency requirements, typed over-limit confirmation-required/retry responses, and error examples in `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T035 [US2] Run purchase/allocator suites in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseTest.php`, `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardInstallmentPurchaseTest.php`, and `/home/ederson/workspace/zunera/zunera-backend/tests/Unit/CreditCards/InstallmentAllocatorTest.php`.

### Frontend for User Story 2

- [ ] T036 [P] [US2] Add purchase submission, fresh-key over-limit confirmation, same-request retry with stable idempotency key, installment preview, category failure, and assigned-statement client tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/services/creditCardService.spec.js` and `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/stores/creditCardStore.spec.js`.
- [ ] T037 [P] [US2] Add currency input, installment preview, closing-date explanation, typed over-limit warning/confirmation with negative availability, and accessible-error tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/components/credit-cards/CreditCardPurchaseForm.spec.js`.
- [ ] T038 [US2] Extend client state for purchases, installments, statement assignment previews, and idempotent feedback in `/home/ederson/workspace/zunera/zunera-frontend/src/services/creditCardService.js` and `/home/ederson/workspace/zunera/zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T039 [US2] Implement purchase entry, over-limit warning/explicit confirmation with negative availability, and installment schedule presentation in `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardPurchaseForm.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/InstallmentSchedule.vue`, and `/home/ederson/workspace/zunera/zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [ ] T040 [US2] Add browser coverage for single/installment purchases, closing-day messaging, over-limit confirmation with a fresh request key, confirmed-request network retry, and unchanged paying-account balance in `/home/ederson/workspace/zunera/zunera-frontend/tests/e2e/credit-cards/credit-card-purchases.spec.js`.

**Checkpoint**: Card spending is recorded once, installments reconcile exactly, and ordinary financial accounts are not charged.

---

## Phase 5: User Story 3 — Review statements and pay them (Priority: P1) 🎯 MVP

**Goal**: Users inspect open and historical statements, make partial or full payments from owned financial accounts, and see accurate obligations and payment history.

**Independent Test**: Close a statement with purchases, pay it in two idempotent payments from an owned account, verify account balance moves once per effective payment, and verify partial, paid, overdue, and invalid-payment states.

### Backend for User Story 3

- [ ] T041 [P] [US3] Write inclusive-closing-day/next-calendar-date finalization plus open, closed, partially paid, paid, overdue, and zero-amount statement coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardStatementTest.php`.
- [ ] T042 [P] [US3] Write 100 representative partial/full settlement, account ownership, intentional overdraft, replay, edit/remove/restore, and payment-history lifecycle cases in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardStatementPaymentTest.php`.
- [ ] T043 [US3] Implement statement refresh, close finalization, state transitions, history, and retrieval in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardStatementService.php`.
- [ ] T044 [US3] Implement payment creation, correction, removal, restoration, account-balance deltas, and double-application prevention in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardStatementPaymentService.php`.
- [ ] T045 [US3] Reuse financial-account balance conventions through a card-payment reconciliation adapter in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardPaymentAccountReconciler.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Services/Transactions/TransactionBalanceReconciler.php`.
- [ ] T046 [US3] Implement payment, correction, removal, and restoration validation in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardStatementPaymentRequest.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Requests/CreditCards/UpdateCreditCardStatementPaymentRequest.php`.
- [ ] T047 [US3] Implement statement, line-item, and payment-history resources in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementResource.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementDetailResource.php`, and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementPaymentResource.php`.
- [ ] T048 [US3] Add statement/payment controller actions and routes in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Controllers/Api/V1/CreditCardStatementController.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Http/Controllers/Api/V1/CreditCardStatementPaymentController.php`, and `/home/ederson/workspace/zunera/zunera-backend/routes/api.php`.
- [ ] T049 [US3] Align statement state, payment, correction, and replay responses in `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T050 [US3] Run statement/payment backend regression suites in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardStatementTest.php` and `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardStatementPaymentTest.php`.

### Frontend for User Story 3

- [ ] T051 [P] [US3] Add statement query, partial-payment same-request retry with stable idempotency key, state, and payment-history reload tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/services/creditCardService.spec.js` and `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/stores/creditCardStore.spec.js`.
- [ ] T052 [P] [US3] Add due-date/status, line-item, partial-total, and account-validation component tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/components/credit-cards/CreditCardStatementPanel.spec.js` and `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/components/credit-cards/CreditCardStatementPaymentDialog.spec.js`.
- [ ] T053 [US3] Extend client state for statements, details, history, and settlement mutations in `/home/ederson/workspace/zunera/zunera-frontend/src/services/creditCardService.js` and `/home/ederson/workspace/zunera/zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T054 [US3] Implement statement list/detail, payment history, and partial-payment flows in `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardStatementList.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardStatementPanel.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardStatementPaymentDialog.vue`, and `/home/ederson/workspace/zunera/zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [ ] T055 [US3] Add end-to-end coverage for partial/full payment, duplicate-submit prevention, account movement, and visible state changes in `/home/ederson/workspace/zunera/zunera-frontend/tests/e2e/credit-cards/credit-card-statement-payments.spec.js`.

**Checkpoint**: Payments settle the obligation and affect only the selected financial account, without creating a second expense.

---

## Phase 6: User Story 4 — Understand cards in budgets, dashboard, and history (Priority: P2)

**Goal**: Users understand card obligations and spending alongside their financial overview without double counting purchase and payment events.

**Independent Test**: Create an open-statement installment and verify it is Expected in its budget month; close it and verify one Realized expense appears in budget, dashboard, and history, while payment adds no category expense.

### Backend for User Story 4

- [ ] T056 [P] [US4] Write 100 representative purchase-and-payment pairs covering pending installments, close realization, category/month allocation, and payment exclusion in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardBudgetIntegrationTest.php`.
- [ ] T057 [P] [US4] Write card-obligation, available-credit, upcoming-due, history, and no-double-count coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardDashboardAndHistoryIntegrationTest.php`.
- [ ] T058 [US4] Integrate expected versus realized installment recognition into `/home/ederson/workspace/zunera/zunera-backend/app/Services/Budgets/BudgetCalculationService.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardBudgetProjectionService.php`.
- [ ] T059 [US4] Integrate card obligations, available credit, and upcoming dues into `/home/ederson/workspace/zunera/zunera-backend/app/Services/Dashboard/DashboardSummaryService.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Services/Dashboard/DashboardUpcomingActivityService.php`, and `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardDashboardProjectionService.php`.
- [ ] T060 [US4] Add recognized card expenses as discriminated history entries and exclude payments from category totals in `/home/ederson/workspace/zunera/zunera-backend/app/Services/FinancialHistory/FinancialHistoryService.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/FinancialHistory/FinancialHistoryResource.php`.
- [ ] T061 [US4] Add dashboard card projection endpoint and route in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Controllers/Api/V1/CreditCardDashboardController.php` and `/home/ederson/workspace/zunera/zunera-backend/routes/api.php`.
- [ ] T062 [US4] Align dashboard summaries and `credit_card_expense` history discriminator in `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T063 [US4] Run integration suites in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardBudgetIntegrationTest.php` and `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardDashboardAndHistoryIntegrationTest.php`.

### Frontend for User Story 4

- [ ] T064 [P] [US4] Add dashboard card-summary service/store tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/services/creditCardService.spec.js` and `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/stores/creditCardStore.spec.js`.
- [ ] T065 [P] [US4] Add obligation/available-credit, upcoming-due, and non-color-only status tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/components/dashboard/CreditCardSummary.spec.js`.
- [ ] T066 [US4] Implement card summary loading and presentation in `/home/ederson/workspace/zunera/zunera-frontend/src/services/creditCardService.js`, `/home/ederson/workspace/zunera/zunera-frontend/src/stores/credit-cards/creditCardStore.js`, `/home/ederson/workspace/zunera/zunera-frontend/src/components/dashboard/CreditCardSummary.vue`, and `/home/ederson/workspace/zunera/zunera-frontend/src/views/dashboard/FinancialDashboardView.vue`.
- [ ] T067 [US4] Update expected/realized card spending and history presentation, and add the recurring-card-unsupported notice with manual-card-purchase guidance, in `/home/ederson/workspace/zunera/zunera-frontend/src/views/budgets/MonthlyBudgetView.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/views/financial-history/FinancialHistoryView.vue`, and `/home/ederson/workspace/zunera/zunera-frontend/src/views/recurring-transactions/RecurringTransactionsListView.vue`.
- [ ] T068 [US4] Add browser coverage for budget recognition, dashboard obligations, history entries, payment non-duplication, and the recurring-card-unsupported notice in `/home/ederson/workspace/zunera/zunera-frontend/tests/e2e/credit-cards/credit-card-financial-overview.spec.js`.

**Checkpoint**: Account balances remain separate from card liabilities; a card expense is recognized exactly once even when later paid.

---

## Phase 7: User Story 5 — Correct purchases and record refunds (Priority: P3)

**Goal**: Users make traceable corrections, cancellations, and refunds while installments, statements, credits, obligations, and available credit reconcile correctly.

**Independent Test**: Correct an open purchase and refund a closed-statement purchase; verify traceable events, updated obligation, oldest-unpaid automatic credit application, and archive cannot hide debt or residual credit.

### Backend for User Story 5

- [ ] T069 [P] [US5] Write pre-close card/category/amount/date/installment-count correction and cancellation coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseCorrectionTest.php`.
- [ ] T070 [P] [US5] Write full/partial refund, post-closing correction credit-event, oldest-unpaid application, available-credit, and archive coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardRefundAndCreditEventTest.php`.
- [ ] T071 [US5] Implement allowed open-purchase corrections, reallocation, immutable history, and closed-purchase rejection in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardPurchaseCorrectionService.php`.
- [ ] T072 [US5] Implement refunds, cancellations, post-closing correction credit events, oldest-unpaid auto-application, and card-credit reconciliation in `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardCreditEventService.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Services/CreditCards/CreditCardObligationReconciler.php`.
- [ ] T073 [US5] Implement correction/credit-event validation in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Requests/CreditCards/UpdateCreditCardPurchaseRequest.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Requests/CreditCards/StoreCreditCardCreditEventRequest.php`.
- [ ] T074 [US5] Implement correction, credit-event, and application-history resources in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardCreditEventResource.php` and `/home/ederson/workspace/zunera/zunera-backend/app/Http/Resources/CreditCards/CreditCardCreditApplicationResource.php`.
- [ ] T075 [US5] Add correction/credit-event controller actions and routes in `/home/ederson/workspace/zunera/zunera-backend/app/Http/Controllers/Api/V1/CreditCardPurchaseController.php`, `/home/ederson/workspace/zunera/zunera-backend/app/Http/Controllers/Api/V1/CreditCardCreditEventController.php`, and `/home/ederson/workspace/zunera/zunera-backend/routes/api.php`.
- [ ] T076 [US5] Align correction, refund, cancellation, credit-application, and conflict responses in `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml`.
- [ ] T077 [US5] Run correction/refund/reconciliation suites in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardPurchaseCorrectionTest.php`, `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardRefundAndCreditEventTest.php`, and `/home/ederson/workspace/zunera/zunera-backend/tests/Unit/CreditCards/CreditCardObligationReconcilerTest.php`.

### Frontend for User Story 5

- [ ] T078 [P] [US5] Add correction, refund, cancellation, application-history, and archive-conflict tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/services/creditCardService.spec.js` and `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/stores/creditCardStore.spec.js`.
- [ ] T079 [P] [US5] Add pre-close correction, post-closing correction-credit-event, refund, card-credit explanation, and invalid-state tests in `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/components/credit-cards/CreditCardCorrectionDialog.spec.js` and `/home/ederson/workspace/zunera/zunera-frontend/tests/unit/components/credit-cards/CreditCardCreditEventDialog.spec.js`.
- [ ] T080 [US5] Extend client state for correction, refund, cancellation, credit-event, and application-history mutations in `/home/ederson/workspace/zunera/zunera-frontend/src/services/creditCardService.js` and `/home/ederson/workspace/zunera/zunera-frontend/src/stores/credit-cards/creditCardStore.js`.
- [ ] T081 [US5] Implement pre-close correction, post-closing correction-credit-event, refund/cancellation, and card-credit history flows in `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardCorrectionDialog.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardCreditEventDialog.vue`, `/home/ederson/workspace/zunera/zunera-frontend/src/components/credit-cards/CreditCardCreditHistory.vue`, and `/home/ederson/workspace/zunera/zunera-frontend/src/views/credit-cards/CreditCardDetailView.vue`.
- [ ] T082 [US5] Add browser coverage for correction, refund, auto-credit application, and archive blocking in `/home/ederson/workspace/zunera/zunera-frontend/tests/e2e/credit-cards/credit-card-corrections-and-refunds.spec.js`.

**Checkpoint**: Corrections/refunds stay auditable, repair the card position, and never silently delete history or duplicate spending.

---

## Phase 8: Polish and release readiness

**Purpose**: Confirm quality, performance, accessibility, documentation, and contract consistency across the feature.

- [ ] T083 [P] Add cross-domain authorization regression coverage in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardAuthorizationTest.php`.
- [ ] T084 [P] Seed 10,000 combined owned financial/card movements; assert card list, statement read, and affected projections each complete within 2 seconds; and capture query-plan evidence before adding indexes in `/home/ederson/workspace/zunera/zunera-backend/tests/Feature/CreditCards/CreditCardQueryPerformanceTest.php` and `/home/ederson/workspace/zunera/zunera-backend/tests/Performance/CreditCards/query-plans.md`.
- [ ] T085 [P] Add responsive theme and keyboard/screen-reader browser checks in `/home/ederson/workspace/zunera/zunera-frontend/tests/e2e/credit-cards/credit-card-accessibility.spec.js`.
- [ ] T086 Validate lifecycle examples and manual acceptance checks in `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/quickstart.md`.
- [ ] T087 Run full backend quality and Credit Cards suites from `/home/ederson/workspace/zunera/zunera-backend/tests/`.
- [ ] T088 Run frontend unit and end-to-end suites from `/home/ederson/workspace/zunera/zunera-frontend/tests/`.
- [ ] T089 Validate final contract at `/home/ederson/workspace/zunera/zunera-specs/specs/010-credit-cards/contracts/credit-cards-api.yaml` with Redocly.
- [ ] T090 Run frontend lint and production-build verification with `npm run lint` and `npm run build` from `/home/ederson/workspace/zunera/zunera-frontend/`.

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
- US4 and US5 may run in parallel after US3 if shared reconciliation and contract edits are coordinated.
- T083–T085 may run in parallel before final validation.

## Parallel example: User Story 2

```bash
Task: "T028 [US2] CreditCardPurchaseTest.php"
Task: "T029 [US2] CreditCardInstallmentPurchaseTest.php"

Task: "T036 [US2] client/store tests"
Task: "T037 [US2] purchase-form component test"
```

## Parallel example: User Story 3

```bash
Task: "T041 [US3] CreditCardStatementTest.php"
Task: "T042 [US3] CreditCardStatementPaymentTest.php"

Task: "T051 [US3] client/store tests"
Task: "T052 [US3] statement component tests"
```

## Implementation strategy

### MVP first (US1 + US2 + US3)

1. Complete Phases 1–2 for authoritative money, ownership, locking, and idempotency rules.
2. Deliver US1, then US2, then US3; validate the P1 flow end to end.
3. Add US4 for visibility and US5 for traceable remediation without changing core recognition.
4. Complete Phase 8 before release; do not defer authorization, idempotency, accessibility, or financial-consistency checks.

### Suggested MVP scope

Phases 1–5 provide the smallest independent release: card management, purchases/installments, statements, and partial/full statement payments. US4 and US5 extend visibility and remediation without changing the established core recognition model.
