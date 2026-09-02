# Tasks: Financial Accounts

**Input**: Design documents from `specs/002-financial-accounts/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/financial-accounts-api.yaml](contracts/financial-accounts-api.yaml), [quickstart.md](quickstart.md)

**Tests**: Required by SQR-004. Write backend feature/unit tests, frontend
service/store/component tests, API contract validation, and Playwright coverage
for critical UI-to-API journeys.

**Organization**: Tasks are grouped by user story so each story can be
implemented and tested independently. Within each story, backend tasks come
before frontend tasks.

**Branch coordination**: Keep `zunera-specs`, `../zunera-backend`, and
`../zunera-frontend` on `002-financial-accounts`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel with other tasks that touch different files and
  do not depend on incomplete work.
- **[Story]**: User story label. Only user story phase tasks include this label.
- Every task includes the concrete file path or repository path to change or
  verify.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm branch alignment and prepare feature-owned folders without
changing behavior.

- [ ] T001 Verify branch `002-financial-accounts` in `/home/ederson/workspace/zunera/zunera-specs/.git/HEAD`, `/home/ederson/workspace/zunera/zunera-backend/.git/HEAD`, and `/home/ederson/workspace/zunera/zunera-frontend/.git/HEAD`
- [ ] T002 [P] Create backend feature directories in `../zunera-backend/app/Data/FinancialAccounts/`, `../zunera-backend/app/Enums/FinancialAccounts/`, `../zunera-backend/app/Exceptions/FinancialAccounts/`, `../zunera-backend/app/Http/Requests/FinancialAccounts/`, `../zunera-backend/app/Http/Resources/FinancialAccounts/`, and `../zunera-backend/app/Services/FinancialAccounts/`
- [ ] T003 [P] Create backend test directories in `../zunera-backend/tests/Feature/FinancialAccounts/` and `../zunera-backend/tests/Unit/FinancialAccounts/`
- [ ] T004 [P] Create frontend feature directories in `../zunera-frontend/src/components/financial-accounts/`, `../zunera-frontend/src/composables/financial-accounts/`, `../zunera-frontend/src/stores/financial-accounts/`, `../zunera-frontend/src/utils/financial-accounts/`, and `../zunera-frontend/src/views/financial-accounts/`
- [ ] T005 [P] Review frontend design constraints for this feature in `docs/design/design-foundation.md`, `docs/design/app-shell.md`, `docs/design/navigation.md`, and `docs/design/components.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Backend domain primitives and API route surface required by all user
stories.

**Critical**: No user story work starts until this phase is complete.

- [ ] T006 Create financial accounts migration in `../zunera-backend/database/migrations/2026_09_02_000000_create_financial_accounts_table.php`
- [ ] T007 [P] Create account type enum in `../zunera-backend/app/Enums/FinancialAccounts/AccountType.php`
- [ ] T008 [P] Create account status enum in `../zunera-backend/app/Enums/FinancialAccounts/AccountStatus.php`
- [ ] T009 [P] Create financial account model in `../zunera-backend/app/Models/FinancialAccount.php`
- [ ] T010 [P] Create financial account factory in `../zunera-backend/database/factories/FinancialAccountFactory.php`
- [ ] T011 [P] Create create/update DTOs in `../zunera-backend/app/Data/FinancialAccounts/CreateFinancialAccountData.php` and `../zunera-backend/app/Data/FinancialAccounts/UpdateFinancialAccountData.php`
- [ ] T012 [P] Create visual identity option catalog in `../zunera-backend/app/Services/FinancialAccounts/FinancialAccountVisualOptions.php`
- [ ] T013 [P] Create state and name conflict exceptions in `../zunera-backend/app/Exceptions/FinancialAccounts/FinancialAccountStateException.php` and `../zunera-backend/app/Exceptions/FinancialAccounts/FinancialAccountNameConflictException.php`
- [ ] T014 Add authenticated financial account route declarations in `../zunera-backend/routes/api.php`

**Checkpoint**: Foundation ready - user story implementation can begin.

---

## Phase 3: User Story 1 - Create and Review Accounts (Priority: P1) MVP

**Goal**: A signed-in user creates an account, sees active accounts, sees each
current balance, and sees combined active-account balance.

**Independent Test**: Sign in, create a checking account named `Conta principal`
with institution `Nubank` and `R$ 1.250,50`, then verify success feedback,
active list visibility, exact current balance, separate institution display, and
combined active balance.

### Tests for User Story 1

- [ ] T015 [P] [US1] Add backend create/list/summary feature tests, including duplicate create suppression for repeated submissions, in `../zunera-backend/tests/Feature/FinancialAccounts/CreateAndListFinancialAccountsTest.php`
- [ ] T016 [P] [US1] Add backend money precision, range, and balance summary unit tests in `../zunera-backend/tests/Unit/FinancialAccounts/FinancialAccountMoneyTest.php`
- [ ] T017 [P] [US1] Add backend normalized active-name uniqueness tests in `../zunera-backend/tests/Unit/FinancialAccounts/FinancialAccountNameNormalizerTest.php`

### Implementation for User Story 1

#### Backend (complete first)

- [ ] T018 [US1] Implement name normalization rules in `../zunera-backend/app/Services/FinancialAccounts/FinancialAccountNameNormalizer.php`
- [ ] T019 [US1] Implement BRL centavo value rules in `../zunera-backend/app/Services/FinancialAccounts/FinancialAccountMoney.php`
- [ ] T020 [US1] Implement create request validation in `../zunera-backend/app/Http/Requests/FinancialAccounts/StoreFinancialAccountRequest.php`
- [ ] T021 [US1] Implement list request validation for status filtering in `../zunera-backend/app/Http/Requests/FinancialAccounts/ListFinancialAccountsRequest.php`
- [ ] T022 [US1] Implement account and summary resources in `../zunera-backend/app/Http/Resources/FinancialAccounts/FinancialAccountResource.php` and `../zunera-backend/app/Http/Resources/FinancialAccounts/FinancialAccountSummaryResource.php`
- [ ] T023 [US1] Implement create, active list, archived list, and summary service behavior in `../zunera-backend/app/Services/FinancialAccounts/FinancialAccountService.php`
- [ ] T024 [US1] Implement list and create controller methods in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialAccountController.php`
- [ ] T025 [US1] Implement active summary controller in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialAccountSummaryController.php`
- [ ] T026 [US1] Verify US1 backend route and test evidence in `../zunera-backend/tests/Feature/FinancialAccounts/CreateAndListFinancialAccountsTest.php`

#### Frontend (after Backend)

- [ ] T027 [P] [US1] Add financial account API service tests in `../zunera-frontend/src/services/__tests__/financialAccountService.spec.js`
- [ ] T028 [P] [US1] Add create/list/summary store tests, including repeated create-action suppression, in `../zunera-frontend/src/stores/financial-accounts/__tests__/financialAccountStore.createList.spec.js`
- [ ] T029 [P] [US1] Add create/list view component tests in `../zunera-frontend/src/views/financial-accounts/__tests__/FinancialAccountsListView.spec.js`
- [ ] T030 [P] [US1] Add create/list Playwright journey, including double-click submit protection, in `../zunera-frontend/e2e/financial-accounts.spec.js`
- [ ] T031 [US1] Implement financial account API service in `../zunera-frontend/src/services/financialAccountService.js`
- [ ] T032 [US1] Implement BRL currency formatting and account options in `../zunera-frontend/src/utils/financial-accounts/currency.js` and `../zunera-frontend/src/utils/financial-accounts/accountOptions.js`
- [ ] T033 [US1] Implement financial account Pinia store in `../zunera-frontend/src/stores/financial-accounts/financialAccountStore.js`
- [ ] T034 [US1] Implement reusable account form for create mode in `../zunera-frontend/src/components/financial-accounts/FinancialAccountForm.vue`
- [ ] T035 [US1] Implement active balance summary and active account list components in `../zunera-frontend/src/components/financial-accounts/FinancialAccountSummary.vue` and `../zunera-frontend/src/components/financial-accounts/FinancialAccountList.vue`
- [ ] T036 [US1] Implement authenticated app shell, page header, primary navigation, and financial accounts list route view in `../zunera-frontend/src/layouts/AppShell.vue`, `../zunera-frontend/src/components/layout/PageHeader.vue`, `../zunera-frontend/src/components/navigation/AppNavigation.vue`, and `../zunera-frontend/src/views/financial-accounts/FinancialAccountsListView.vue`
- [ ] T037 [US1] Wire authenticated shell routing, protected financial accounts route, and primary navigation entry in `../zunera-frontend/src/App.vue` and `../zunera-frontend/src/router/index.js`

**Checkpoint**: User Story 1 is independently usable as the MVP.

---

## Phase 4: User Story 2 - View and Update Account Details (Priority: P2)

**Goal**: A signed-in user opens one owned account, reviews all stored details,
updates editable fields, and receives clear validation or success feedback.

**Independent Test**: Sign in, open an owned account detail view, update account
name, account type, institution, color, icon, and eligible initial balance, then
verify ownership protection, durable success feedback, exact balances, and
initial-balance lock behavior after movements exist.

### Tests for User Story 2

- [ ] T038 [P] [US2] Add backend detail/update feature tests in `../zunera-backend/tests/Feature/FinancialAccounts/ViewAndUpdateFinancialAccountsTest.php`
- [ ] T039 [P] [US2] Add backend initial-balance lock unit tests in `../zunera-backend/tests/Unit/FinancialAccounts/FinancialAccountUpdateRulesTest.php`
- [ ] T040 [P] [US2] Add frontend detail/update API service and store tests in `../zunera-frontend/src/services/__tests__/financialAccountService.spec.js` and `../zunera-frontend/src/stores/financial-accounts/__tests__/financialAccountStore.detailUpdate.spec.js`
- [ ] T041 [P] [US2] Add frontend detail/update view tests in `../zunera-frontend/src/views/financial-accounts/__tests__/FinancialAccountDetailView.spec.js`

### Implementation for User Story 2

#### Backend (complete first)

- [ ] T042 [US2] Implement update request validation in `../zunera-backend/app/Http/Requests/FinancialAccounts/UpdateFinancialAccountRequest.php`
- [ ] T043 [US2] Extend detail, update, ownership, name conflict, and initial-balance lock rules in `../zunera-backend/app/Services/FinancialAccounts/FinancialAccountService.php`
- [ ] T044 [US2] Extend get and update controller behavior in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialAccountController.php`
- [ ] T045 [US2] Verify US2 backend route and test evidence in `../zunera-backend/tests/Feature/FinancialAccounts/ViewAndUpdateFinancialAccountsTest.php`

#### Frontend (after Backend)

- [ ] T046 [P] [US2] Add detail/update Playwright journey in `../zunera-frontend/e2e/financial-accounts.spec.js`
- [ ] T047 [US2] Extend financial account API service with detail and update methods in `../zunera-frontend/src/services/financialAccountService.js`
- [ ] T048 [US2] Extend Pinia store with selected detail, update action, and server validation mapping in `../zunera-frontend/src/stores/financial-accounts/financialAccountStore.js`
- [ ] T049 [US2] Extend account form for update mode and locked initial-balance feedback in `../zunera-frontend/src/components/financial-accounts/FinancialAccountForm.vue`
- [ ] T050 [US2] Implement account detail view in `../zunera-frontend/src/views/financial-accounts/FinancialAccountDetailView.vue`
- [ ] T051 [US2] Add protected account detail route in `../zunera-frontend/src/router/index.js`

**Checkpoint**: User Stories 1 and 2 work independently.

---

## Phase 5: User Story 3 - Archive and Restore Accounts (Priority: P3)

**Goal**: A signed-in user archives unused accounts, keeps historical access,
and restores archived accounts when state rules allow.

**Independent Test**: Sign in, archive an active account including the last
active account case, verify active summary becomes zero when appropriate, view
archived accounts, restore an archived account, and verify state conflict
feedback for already-archived, already-active, or duplicate-name restore cases.

### Tests for User Story 3

- [ ] T052 [P] [US3] Add backend archive/restore feature tests, including repeated archive and repeated restore suppression, in `../zunera-backend/tests/Feature/FinancialAccounts/ArchiveAndRestoreFinancialAccountsTest.php`
- [ ] T053 [P] [US3] Add backend lifecycle state unit tests in `../zunera-backend/tests/Unit/FinancialAccounts/FinancialAccountLifecycleTest.php`
- [ ] T054 [P] [US3] Add frontend archive/restore API service and store tests in `../zunera-frontend/src/services/__tests__/financialAccountService.spec.js` and `../zunera-frontend/src/stores/financial-accounts/__tests__/financialAccountStore.lifecycle.spec.js`
- [ ] T055 [P] [US3] Add frontend lifecycle view tests in `../zunera-frontend/src/views/financial-accounts/__tests__/ArchivedFinancialAccountsView.spec.js`

### Implementation for User Story 3

#### Backend (complete first)

- [ ] T056 [US3] Extend lifecycle archive and restore rules in `../zunera-backend/app/Services/FinancialAccounts/FinancialAccountService.php`
- [ ] T057 [US3] Extend archive and restore controller behavior in `../zunera-backend/app/Http/Controllers/Api/V1/FinancialAccountController.php`
- [ ] T058 [US3] Verify US3 backend route and test evidence in `../zunera-backend/tests/Feature/FinancialAccounts/ArchiveAndRestoreFinancialAccountsTest.php`

#### Frontend (after Backend)

- [ ] T059 [P] [US3] Add archive/restore Playwright journey, including repeated lifecycle-action protection, in `../zunera-frontend/e2e/financial-accounts.spec.js`
- [ ] T060 [US3] Extend financial account API service with archive and restore methods in `../zunera-frontend/src/services/financialAccountService.js`
- [ ] T061 [US3] Extend Pinia store with archive, restore, archived list, and state conflict handling in `../zunera-frontend/src/stores/financial-accounts/financialAccountStore.js`
- [ ] T062 [US3] Implement lifecycle confirmation dialog in `../zunera-frontend/src/components/financial-accounts/FinancialAccountLifecycleDialog.vue`
- [ ] T063 [US3] Implement archived account list view in `../zunera-frontend/src/views/financial-accounts/ArchivedFinancialAccountsView.vue`
- [ ] T064 [US3] Add archived accounts route and navigation path in `../zunera-frontend/src/router/index.js`

**Checkpoint**: All user stories work independently.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Verify contracts, full suites, visual states, and release evidence
across all stories.

- [ ] T065 [P] Lint OpenAPI contract in `specs/002-financial-accounts/contracts/financial-accounts-api.yaml`
- [ ] T066 [P] Run focused backend feature and unit suites in `../zunera-backend/tests/Feature/FinancialAccounts/` and `../zunera-backend/tests/Unit/FinancialAccounts/`
- [ ] T067 [P] Run full backend suite and style check in `../zunera-backend/tests/` and `../zunera-backend/app/`
- [ ] T068 [P] Run frontend unit tests for financial account service, store, and views in `../zunera-frontend/src/services/__tests__/`, `../zunera-frontend/src/stores/financial-accounts/__tests__/`, and `../zunera-frontend/src/views/financial-accounts/__tests__/`
- [ ] T069 [P] Run frontend build verification in `../zunera-frontend/src/`
- [ ] T070 [P] Run Playwright financial account journey in `../zunera-frontend/e2e/financial-accounts.spec.js`
- [ ] T071 Validate app shell, primary navigation, Light, Dark, System, 320px viewport, 200 percent zoom, keyboard-only navigation, and accessible labels against `docs/design/design-foundation.md`, `docs/design/app-shell.md`, and `docs/design/navigation.md`
- [ ] T072 Record quickstart release evidence in `specs/002-financial-accounts/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks every user story.
- **Phase 3 US1**: Depends on Phase 2 and is the MVP.
- **Phase 4 US2**: Depends on Phase 2, but should be integrated after US1 for
  the normal product path.
- **Phase 5 US3**: Depends on Phase 2, but should be integrated after US1 and
  US2 so lifecycle actions have list and detail surfaces.
- **Phase 6 Polish**: Depends on all intended user stories being complete.

### User Story Dependencies

- **US1 Create and Review Accounts**: First deliverable after foundation; no
  dependency on US2 or US3.
- **US2 View and Update Account Details**: Can be developed after foundation;
  uses the shared account model, service, resources, API service, and store.
- **US3 Archive and Restore Accounts**: Can be developed after foundation; uses
  the shared account lifecycle fields, service, resources, API service, and
  store.

### Within Each User Story

- Backend tests first, then backend implementation.
- Backend API contract, authorization, validation, and tests before frontend
  implementation.
- Frontend service and store before route views and components that consume
  them.
- Playwright coverage validates the completed UI-to-API journey for each story.

---

## Parallel Opportunities

- Setup tasks T002, T003, T004, and T005 can run in parallel.
- Foundational enum, model, factory, DTO, visual option, and exception tasks
  T007 through T013 can run in parallel after T006 is understood.
- Test-writing tasks inside each user story can run in parallel because they
  target different files.
- Frontend service/store tests and view tests can run in parallel after backend
  behavior is known.
- Full verification tasks T065 through T070 can run in parallel after all stories
  are implemented.

## Parallel Example: User Story 1

```bash
# Backend tests can be authored together:
Task: "T015 CreateAndListFinancialAccountsTest.php"
Task: "T016 FinancialAccountMoneyTest.php"
Task: "T017 FinancialAccountNameNormalizerTest.php"

# Frontend tests can be authored together after backend contract is verified:
Task: "T027 financialAccountService.spec.js"
Task: "T028 financialAccountStore.createList.spec.js"
Task: "T029 FinancialAccountsListView.spec.js"
Task: "T030 financial-accounts.spec.js"
```

## Parallel Example: User Story 2

```bash
Task: "T038 ViewAndUpdateFinancialAccountsTest.php"
Task: "T039 FinancialAccountUpdateRulesTest.php"
Task: "T040 financialAccountStore.detailUpdate.spec.js"
Task: "T041 FinancialAccountDetailView.spec.js"
```

## Parallel Example: User Story 3

```bash
Task: "T052 ArchiveAndRestoreFinancialAccountsTest.php"
Task: "T053 FinancialAccountLifecycleTest.php"
Task: "T054 financialAccountStore.lifecycle.spec.js"
Task: "T055 ArchivedFinancialAccountsView.spec.js"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup.
2. Complete Phase 2 foundation.
3. Complete US1 backend tests and implementation.
4. Complete US1 frontend tests and implementation.
5. Stop and validate US1 independently with backend tests, frontend tests,
   Playwright, and the quickstart smoke path.

### Incremental Delivery

1. Deliver US1 as the usable account creation, active list, and active balance
   summary increment.
2. Add US2 detail and update behavior without changing US1 contracts.
3. Add US3 archive and restore behavior without permanent deletion.
4. Run Phase 6 verification before review.

### Team Parallel Strategy

1. Complete setup and foundation together.
2. One backend developer can implement service/controller work while another
   writes feature/unit tests for the same story.
3. Frontend work starts after backend contract and tests pass for that story.
4. Different developers may work on US2 and US3 after foundation, but integrate
   in priority order: US1, then US2, then US3.

## Notes

- Do not add new packages without explicit approval.
- Account name is the user-defined label; `institution_name` is optional
  institution text and remains separate.
- Money is signed integer BRL centavos; do not use floating-point arithmetic for
  balances.
- Permanent deletion is out of scope; archive is the lifecycle mechanism.
- Archived accounts are excluded from active combined balance and normal
  new-operation choices.
