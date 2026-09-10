# Tasks: Category Management

**Input**: Design documents from `specs/003-category-management/`  
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md),
[research.md](research.md), [data-model.md](data-model.md),
[categories-api.yaml](contracts/categories-api.yaml), and
[quickstart.md](quickstart.md)

**Tests**: Tests are required by SQR-004 and the constitution. Add Laravel
feature/unit tests, frontend service/store/view tests, contract linting, and
isolated Playwright journeys with accessible selectors.

**Organization**: Backend tasks for each story complete before that story's
frontend tasks. Stories are independently testable at their checkpoints.

**Branch coordination**: Before work, confirm `003-category-management` in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Different files and no incomplete-task dependency.
- **[Story]**: User story traceability label.
- Every task includes its exact target path.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm shared feature scope and prepare focused domain locations.

- [X] T001 Verify branch `003-category-management` in `.`,
  `../zunera-backend`, and `../zunera-frontend` before editing application
  files.
- [X] T002 Create category domain folders in
  `../zunera-backend/app/{Data,Enums,Exceptions,Http/Requests,Http/Resources,Services}/Categories/`
  and `../zunera-backend/tests/{Feature,Unit}/Categories/`.
- [X] T003 [P] Create category feature folders in
  `../zunera-frontend/src/{components,stores,utils,views}/categories/` and
  `../zunera-frontend/e2e/` without adding dependencies.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish the shared persisted category domain before any user
story. All user-story work waits for this phase.

- [X] T004 [P] Create `CategoryOrigin`, `CategoryClassification`, and
  `CategoryStatus` enums in `../zunera-backend/app/Enums/Categories/`.
- [X] T005 [P] Create normalized-name and permitted visual-option services in
  `../zunera-backend/app/Services/Categories/CategoryNameNormalizer.php` and
  `../zunera-backend/app/Services/Categories/CategoryVisualOptions.php`.
- [X] T006 Create category persistence, system/personal ownership fields,
  active-name integrity, archive fields, and historical-use lock in
  `../zunera-backend/database/migrations/*_create_categories_table.php`.
- [X] T007 [P] Create `Category` factory with system, personal, active,
  archived, and used-category states in
  `../zunera-backend/database/factories/CategoryFactory.php`.
- [X] T008 Create idempotent 16-default category seeding in
  `../zunera-backend/database/seeders/CategorySeeder.php` and register it in
  `../zunera-backend/database/seeders/DatabaseSeeder.php`.
- [X] T009 Create guarded casts, ownership relation, lifecycle helpers, and
  normalization hooks in `../zunera-backend/app/Models/Category.php`.
- [X] T010 [P] Create create/update DTOs in
  `../zunera-backend/app/Data/Categories/CreateCategoryData.php` and
  `../zunera-backend/app/Data/Categories/UpdateCategoryData.php`.
- [X] T011 [P] Create typed name-conflict and state exceptions in
  `../zunera-backend/app/Exceptions/Categories/CategoryNameConflictException.php`
  and `../zunera-backend/app/Exceptions/Categories/CategoryStateException.php`.
- [X] T012 Implement shared owner lookup, system-default lookup, conflict
  checks, and safe exception mapping skeleton in
  `../zunera-backend/app/Services/Categories/CategoryService.php` and
  `../zunera-backend/app/Http/Controllers/Api/V1/CategoryController.php`.
- [X] T013 [P] Add foundational normalization, permitted visual-choice, model,
  factory, and seeder tests in
  `../zunera-backend/tests/Unit/Categories/CategoryNameNormalizerTest.php`,
  `../zunera-backend/tests/Unit/Categories/CategoryVisualOptionsTest.php`, and
  `../zunera-backend/tests/Feature/Categories/CategoryDefaultsTest.php`.
- [X] T014 Run migration and foundational category tests from
  `../zunera-backend` before beginning user stories.

**Checkpoint**: Persisted system defaults, personal ownership boundary, shared
validation helpers, and test fixtures are ready.

---

## Phase 3: User Story 1 - Browse Categories (Priority: P1) 🎯 MVP

**Goal**: Authenticated users can see active income/expense system defaults and
their own active personal categories, without seeing another user's categories.

**Independent Test**: A signed-in user with no personal categories sees all 16
system defaults with origin/classification; a second user's personal category is
absent; the active category page has readable loading, empty, error, and success
states.

### Tests for User Story 1

- [X] T015 [P] [US1] Add authenticated list, default-origin, active/archived
  filtering, and cross-user privacy coverage in
  `../zunera-backend/tests/Feature/Categories/ListCategoriesTest.php`.
- [X] T016 [P] [US1] Add service-level active-list composition and safe
  owner/system lookup coverage in
  `../zunera-backend/tests/Unit/Categories/CategoryListingTest.php`.

### Implementation for User Story 1

#### Backend (complete first)

- [X] T017 [US1] Add validated `status` filtering in
  `../zunera-backend/app/Http/Requests/Categories/ListCategoriesRequest.php`.
- [X] T018 [US1] Implement stable category resource serialization in
  `../zunera-backend/app/Http/Resources/Categories/CategoryResource.php`.
- [X] T019 [US1] Implement active default-plus-owned-personal and
  archived-owned-personal list behavior in
  `../zunera-backend/app/Services/Categories/CategoryService.php`.
- [X] T020 [US1] Implement the authenticated category index action in
  `../zunera-backend/app/Http/Controllers/Api/V1/CategoryController.php`.
- [X] T021 [US1] Register protected `GET /api/v1/categories` behavior in
  `../zunera-backend/routes/api.php` according to
  `specs/003-category-management/contracts/categories-api.yaml`.
- [X] T022 [US1] Run focused list/default tests and verify the published list
  contract in
  `../zunera-backend/tests/Feature/Categories/ListCategoriesTest.php`,
  `../zunera-backend/tests/Unit/Categories/CategoryListingTest.php`, and
  `specs/003-category-management/contracts/categories-api.yaml`.

#### Frontend (after Backend)

- [X] T023 [P] [US1] Add list-request and normalized-error tests in
  `../zunera-frontend/src/services/__tests__/categoryService.spec.js`.
- [X] T024 [P] [US1] Add active/archived collection, loading, and safe-error
  store tests in
  `../zunera-frontend/src/stores/categories/__tests__/categoryStore.list.spec.js`.
- [X] T025 [P] [US1] Add default-origin marker, classification label,
  responsive table/card, and state rendering tests in
  `../zunera-frontend/src/components/categories/__tests__/CategoryList.spec.js`.
- [X] T026 [US1] Implement list requests against the verified contract in
  `../zunera-frontend/src/services/categoryService.js`.
- [X] T027 [US1] Implement setup-style active/archived category state and list
  reconciliation in `../zunera-frontend/src/stores/categories/categoryStore.js`.
- [X] T028 [US1] Implement permitted labels, default visual mappings, and
  non-semantic category-color helpers in
  `../zunera-frontend/src/utils/categories/categoryOptions.js`.
- [X] T029 [US1] Implement accessible category list with a textual “System
  default” marker, no default mutation affordances, and loading/empty/error
  states in `../zunera-frontend/src/components/categories/CategoryList.vue`.
- [X] T030 [US1] Add authenticated active and archived category routes in
  `../zunera-frontend/src/router/index.js`.
- [X] T031 [US1] Add category navigation entries with stable route names and
  accessible labels in `../zunera-frontend/src/layouts/AppShell.vue`.
- [X] T032 [US1] Compose active-category fetching, page header, and list state
  in `../zunera-frontend/src/views/categories/CategoriesListView.vue`.
- [X] T033 [US1] Compose archived-category fetching and safe empty/error state
  in `../zunera-frontend/src/views/categories/ArchivedCategoriesView.vue`.
- [X] T034 [US1] Add active-list/default-origin/privacy and archived-list UI
  coverage in `../zunera-frontend/src/views/categories/__tests__/`.
- [X] T035 [US1] Add isolated active and archived category list journeys with
  accessible selectors in `../zunera-frontend/e2e/categories.spec.js`.
- [X] T036 [US1] Run focused list/store/view tests in
  `../zunera-frontend/src/services/__tests__/categoryService.spec.js`,
  `../zunera-frontend/src/stores/categories/__tests__/categoryStore.list.spec.js`,
  and `../zunera-frontend/src/views/categories/__tests__/` before US2.

**Checkpoint**: Browsing defaults and owned categories works independently in
the authenticated shell. This is the MVP.

---

## Phase 4: User Story 2 - Create and Update Personal Categories (Priority: P1)

**Goal**: Users create and update their own personal categories with validated
name, classification, color, and icon choices while preserving history.

**Independent Test**: A user creates and updates a valid unused personal
category; duplicates, invalid choices, another user's ID, system-default
mutations, and post-use classification changes receive safe feedback.

### Tests for User Story 2

- [X] T037 [P] [US2] Add create/update, input validation, normalized duplicate,
  system-default collision, ownership, and classification-lock feature tests in
  `../zunera-backend/tests/Feature/Categories/CreateAndUpdateCategoriesTest.php`.
- [X] T038 [P] [US2] Add personal create/update conflict and used-category
  classification-lock unit tests in
  `../zunera-backend/tests/Unit/Categories/CategoryUpdateRulesTest.php`.

### Implementation for User Story 2

#### Backend (complete first)

- [X] T039 [P] [US2] Implement create-request authorization, normalization,
  classification, and visual-choice validation in
  `../zunera-backend/app/Http/Requests/Categories/StoreCategoryRequest.php`.
- [X] T040 [P] [US2] Implement update-request authorization, partial-field,
  classification-lock, and visual-choice validation in
  `../zunera-backend/app/Http/Requests/Categories/UpdateCategoryRequest.php`.
- [X] T041 [US2] Implement owned personal-category create, read, and update
  rules, default conflict rejection, and classification locking in
  `../zunera-backend/app/Services/Categories/CategoryService.php`.
- [X] T042 [US2] Implement create, show, and update resource actions with typed
  404/409/422 responses in
  `../zunera-backend/app/Http/Controllers/Api/V1/CategoryController.php`.
- [X] T043 [US2] Register protected create, show, and update routes in
  `../zunera-backend/routes/api.php` according to
  `specs/003-category-management/contracts/categories-api.yaml`.
- [X] T044 [US2] Run focused create/update backend tests and lint the verified
  category contract at
  `specs/003-category-management/contracts/categories-api.yaml`.

#### Frontend (after Backend)

- [X] T045 [P] [US2] Extend service tests for create, show, update, CSRF, field
  errors, and 404/409 codes in
  `../zunera-frontend/src/services/__tests__/categoryService.spec.js`.
- [X] T046 [P] [US2] Add create/update, duplicate-submit, reconciliation, and
  conflict-code store tests in
  `../zunera-frontend/src/stores/categories/__tests__/categoryStore.createUpdate.spec.js`.
- [X] T047 [P] [US2] Add form validation, semantic-color separation, and locked
  classification tests in
  `../zunera-frontend/src/components/categories/__tests__/CategoryForm.spec.js`.
- [X] T048 [US2] Extend create, show, and update transport calls in
  `../zunera-frontend/src/services/categoryService.js`.
- [X] T049 [US2] Extend create/update actions, field-error mapping, and
  duplicate-submit protection in
  `../zunera-frontend/src/stores/categories/categoryStore.js`.
- [X] T050 [US2] Implement top-labeled `ElForm` category create/edit dialog,
  permitted color/icon choices, and locked-classification help text in
  `../zunera-frontend/src/components/categories/CategoryForm.vue`.
- [X] T051 [US2] Add owned-personal edit actions and immutable/default state
  affordances to `../zunera-frontend/src/components/categories/CategoryList.vue`.
- [X] T052 [US2] Compose create/edit dialogs, durable success feedback, and
  error recovery in
  `../zunera-frontend/src/views/categories/CategoriesListView.vue`.
- [X] T053 [US2] Add create, update, duplicate, read-only-default, and
  classification-lock journeys in
  `../zunera-frontend/e2e/categories.spec.js`.
- [X] T054 [US2] Run focused create/update tests in
  `../zunera-frontend/src/services/__tests__/categoryService.spec.js`,
  `../zunera-frontend/src/stores/categories/__tests__/categoryStore.createUpdate.spec.js`,
  and `../zunera-frontend/src/components/categories/__tests__/CategoryForm.spec.js`
  before US3.

**Checkpoint**: Personal category creation and permitted updates work without
changing default or historic financial meaning.

---

## Phase 5: User Story 3 - Archive and Restore Personal Categories (Priority: P2)

**Goal**: Users archive and restore their own categories while preserving
historical associations and keeping archived categories out of new selections.

**Independent Test**: A user archives an owned active personal category, finds
it only in archived management/history contexts, restores it when no conflict
exists, and receives clear feedback for invalid lifecycle requests.

### Tests for User Story 3

- [X] T055 [P] [US3] Add archive/restore ownership, system-read-only, state,
  normalized restore-conflict, used-category lock, and permanent-delete
  route-absence feature coverage in
  `../zunera-backend/tests/Feature/Categories/ArchiveAndRestoreCategoriesTest.php`.
- [X] T056 [P] [US3] Add active/archived transition and restore-conflict unit
  coverage in
  `../zunera-backend/tests/Unit/Categories/CategoryLifecycleTest.php`.

### Implementation for User Story 3

#### Backend (complete first)

- [X] T057 [US3] Implement owned personal archive/restore transitions that
  preserve used-category records, plus state conflict codes, in
  `../zunera-backend/app/Services/Categories/CategoryService.php`.
- [X] T058 [US3] Implement archive and restore actions in
  `../zunera-backend/app/Http/Controllers/Api/V1/CategoryController.php`.
- [X] T059 [US3] Register protected archive and restore routes in
  `../zunera-backend/routes/api.php` according to
  `specs/003-category-management/contracts/categories-api.yaml`.
- [X] T060 [US3] Run focused lifecycle backend tests in
  `../zunera-backend/tests/Feature/Categories/ArchiveAndRestoreCategoriesTest.php`
  and `../zunera-backend/tests/Unit/Categories/CategoryLifecycleTest.php`; verify
  archived categories are excluded from normal active-list responses.

#### Frontend (after Backend)

- [X] T061 [P] [US3] Extend archive/restore service and normalized conflict
  tests in `../zunera-frontend/src/services/__tests__/categoryService.spec.js`.
- [X] T062 [P] [US3] Add lifecycle reconciliation and repeated-action store
  tests in
  `../zunera-frontend/src/stores/categories/__tests__/categoryStore.lifecycle.spec.js`.
- [X] T063 [P] [US3] Add confirmation-copy, focus-return, disabled-state, and
  no-permanent-delete-action tests in
  `../zunera-frontend/src/components/categories/__tests__/CategoryLifecycleDialog.spec.js`.
- [X] T064 [US3] Extend archive/restore transport calls in
  `../zunera-frontend/src/services/categoryService.js`.
- [X] T065 [US3] Extend archive/restore actions and active/archived collection
  reconciliation in `../zunera-frontend/src/stores/categories/categoryStore.js`.
- [X] T066 [US3] Implement confirmation and lifecycle feedback dialog in
  `../zunera-frontend/src/components/categories/CategoryLifecycleDialog.vue`.
- [X] T067 [US3] Add owned archive/restore actions, unavailable-action
  explanation, and archived markers in
  `../zunera-frontend/src/components/categories/CategoryList.vue`.
- [X] T068 [US3] Compose archive confirmation, restore feedback, and archived
  list actions in `../zunera-frontend/src/views/categories/CategoriesListView.vue`
  and `../zunera-frontend/src/views/categories/ArchivedCategoriesView.vue`.
- [X] T069 [US3] Add archive, restore, repeated-state-conflict, and
  used-category lifecycle journeys in
  `../zunera-frontend/e2e/categories.spec.js`.
- [X] T070 [US3] Run focused lifecycle tests in
  `../zunera-frontend/src/stores/categories/__tests__/categoryStore.lifecycle.spec.js`
  and
  `../zunera-frontend/src/components/categories/__tests__/CategoryLifecycleDialog.spec.js`
  before cross-cutting verification.

**Checkpoint**: Lifecycle behavior preserves personal category history and
keeps new-transaction choices focused.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Complete full-suite, contract, accessibility, responsive, and
documentation evidence across delivered stories.

- [X] T071 [P] Lint the final contract in
  `specs/003-category-management/contracts/categories-api.yaml` with Redocly.
- [X] T072 [P] Run full backend tests and Pint from `../zunera-backend` after
  all category changes.
- [X] T073 [P] Run frontend unit tests and production build from
  `../zunera-frontend` after all category changes.
- [X] T074 Run category Playwright journeys, including Light/Dark/System,
  320px, 200% zoom, keyboard focus, and mobile navigation, in
  `../zunera-frontend/e2e/categories.spec.js`.
- [X] T075 Record final verification evidence and any environment-limited checks
  in `specs/003-category-management/quickstart.md`.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies.
- **Foundational (Phase 2)**: Depends on T001–T003 and blocks all stories.
- **US1 (Phase 3)**: Depends on Phase 2. It establishes the browsable category
  lists required to verify later mutations.
- **US2 (Phase 4)**: Depends on the verified US1 list contract so creation and
  update results can be reconciled and displayed.
- **US3 (Phase 5)**: Depends on US1 list presentation and US2 personal-category
  creation; lifecycle controls reuse those verified boundaries.
- **Polish (Phase 6)**: Depends on completed desired user stories.

### Within Each User Story

1. Write focused backend tests before behavior changes.
2. Implement backend validation, service rules, routes, resources, and tests.
3. Run focused backend tests and verify the contract.
4. Write frontend service/store/component tests, then implement services, state,
   routes, and UI.
5. Run focused frontend tests and isolated Playwright journey.

### Parallel Opportunities

- T004, T005, T007, T010, T011, and T013 work on independent foundational
  files after their stated prerequisites.
- In each story, `[P]` test tasks can be authored in parallel because they own
  distinct files and mock the contract boundary.
- T023–T025, T045–T047, and T061–T063 are parallel frontend test opportunities
  after their corresponding backend contract checkpoint.
- Cross-cutting T071–T073 can run in parallel once implementation completes;
  T074 follows their stable build/test baseline.

## Parallel Example: User Story 2

```text
T037  Backend create/update feature coverage
T038  Backend update-rule unit coverage

T039  Store request validation
T040  Store update request validation

T045  Frontend service tests
T046  Frontend store tests
T047  Category form tests
```

## Implementation Strategy

### MVP First

1. Complete Setup and Foundational phases.
2. Deliver US1 backend list/default behavior and verify it.
3. Deliver US1 frontend category browsing and verify it in the authenticated
   shell.
4. Stop at the US1 checkpoint for an independently demonstrable default
   category-management MVP.

### Incremental Delivery

1. US1 adds visibility of system defaults and personal category availability.
2. US2 adds personal category creation and safe updates.
3. US3 adds lifecycle control without breaking historical associations.
4. Phase 6 validates contract, full suites, responsive themes, accessibility,
   and release evidence.

## Notes

- `[P]` means the listed task owns different files and may proceed in parallel.
- Each user-story task has a `[US#]` traceability label.
- Do not start frontend work for a story before that story's backend contract,
  authorization, validation, and tests pass.
- No task adds a package, transaction entry, reporting, budgets, nested
  categories, sharing, custom classifications, or permanent deletion.
