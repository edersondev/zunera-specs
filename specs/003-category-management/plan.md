# Implementation Plan: Category Management

**Branch**: `003-category-management` | **Date**: 2026-09-04 | **Spec**:
[spec.md](spec.md)  
**Input**: Feature specification from
`specs/003-category-management/spec.md`

**Branch Coordination**: `003-category-management` is checked out in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

**Note**: Planning ends after Phase 1 design. Task generation is the
responsibility of `/speckit-tasks`.

## Summary

Build authenticated category management with system defaults plus user-owned
personal categories. The backend first provides protected category contracts,
default seeding, normalized conflict checks, lifecycle behavior, historical-use
protection, and tests. The frontend then consumes that contract through the
authenticated app shell with responsive, theme-aware category lists, forms, and
lifecycle feedback.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13.17+; JavaScript with Vue 3.5.40  
**Primary Dependencies**: Laravel Sanctum 4, API Resources/Form Requests;
Axios 1.19, Pinia 4.0.2, Vue Router 5.2, Element Plus 2.14.5, Tailwind CSS
4.3, Vite 8.1  
**Storage**: Existing relational application persistence; persisted global
defaults and personal category records  
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.62, Redocly
OpenAPI lint  
**Target Platform**: Authenticated Zunera web application  
**Project Type**: Full-stack web feature across specs, backend API, and
frontend SPA  
**Performance Goals**: Category list and completed create/update/lifecycle
feedback are interactive within normal authenticated-page expectations; no bulk
or reporting workload is added  
**Constraints**: No new packages; protected routes; service-owned business
rules; personal ownership isolation; no permanent deletion; colors use only the
Design Foundation category palette, never income/expense or state semantics  
**Scale/Scope**: System default set of 16 categories plus personal categories
owned by one user; no transaction entry, reporting, budgets, nested categories,
sharing, or custom classifications

## Delivery Scope and Order

Document both scopes in this exact order:

1. **Backend** — Category API contract, persistence, system-default seeding,
   authorization, validation, service-layer lifecycle and conflict rules, and
   backend tests in `../zunera-backend`.
2. **Frontend** — Vue category views, Axios service, Pinia state, Element Plus
   controls, design-compliant navigation, and frontend tests in
   `../zunera-frontend`.

Frontend implementation MUST start only after the relevant backend API contract,
authorization, validation, and tests are complete.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] API boundaries use authenticated middleware, Form Requests, API
      Resources, and no upload handling.
- [x] Backend contract, authorization, validation, and tests precede frontend
      work in `../zunera-frontend`.
- [x] Branch name matches across all three repositories:
      `003-category-management`.
- [x] Category business rules live in a focused service. Create/update payloads
      justify DTOs; simple category access does not justify a repository.
- [x] Frontend uses JavaScript `<script setup>`, Axios service access, focused
      Pinia state, and Element Plus standard controls.
- [x] Backend behavior/API contract coverage and isolated Playwright critical
      journeys are planned.
- [x] Ownership, safe feedback, no secrets/uploads, and scope limits are
      recorded.
- [x] No constitution exception is required.

## Project Structure

### Documentation (this feature)

```text
specs/003-category-management/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/requirements.md
├── contracts/categories-api.yaml
└── tasks.md              # Created by /speckit-tasks, not this command
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/
│   ├── Data/Categories/
│   ├── Enums/Categories/
│   ├── Exceptions/Categories/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/Categories/
│   │   └── Resources/Categories/
│   ├── Models/Category.php
│   └── Services/Categories/
├── database/
│   ├── factories/CategoryFactory.php
│   ├── migrations/
│   └── seeders/
├── routes/api.php
└── tests/
    ├── Feature/Categories/
    └── Unit/Categories/

../zunera-frontend/
├── e2e/categories.spec.js
├── src/
│   ├── components/categories/
│   ├── router/index.js
│   ├── services/categoryService.js
│   ├── stores/categories/categoryStore.js
│   ├── utils/categories/categoryOptions.js
│   └── views/categories/
└── tests through colocated `__tests__/` folders
```

**Structure Decision**: Follow the completed financial-account feature's
vertical-slice layout. Backend files remain grouped by domain and establish the
contract first. Frontend route views compose small category list, form, and
lifecycle-dialog components; transport behavior stays in the service and shared
cross-view state stays in the feature store.

## Phase 0 Research Decisions

- Persist the 16 required system defaults with `origin=system`, no user owner,
  and active/read-only behavior. This gives every authenticated user a stable,
  queryable default set and makes origin explicit in responses. A static,
  non-persisted list was rejected because it complicates contract consistency,
  default identity, and future transaction associations.
- Persist personal categories with `origin=personal` and exactly one owner.
  Active list results combine global active defaults with the signed-in user's
  active personal categories; archived results contain only that user's
  archived personal categories.
- Use `income` and `expense` enums only. A personal category classification may
  change before use; `has_financial_transactions` locks it after first use.
  This mirrors the completed financial-account feature's forward-compatible
  movement lock because transaction records do not yet exist in the product.
- This feature creates no transaction records. Category tests use used-category
  fixtures and `has_financial_transactions` to prove the lock and no-delete
  behavior; a future transaction feature must replace or extend that coverage
  with real category-association retention tests.
- Enforce normalized active personal-name uniqueness at persistence level by
  owner, classification, and active state. The category service also rejects a
  collision with a system default, which cannot be represented by a
  user-scoped uniqueness rule alone.
- Use active/archived lifecycle with `archived_at`; do not provide deletion.
  A repeated lifecycle action or restore/name conflict returns a typed state or
  conflict response. Other users' personal categories return privacy-safe 404;
  known system defaults reject mutation with a clear 409 read-only state code.
- Permit only Design Foundation chart-palette identifiers and feature-approved
  icon identifiers. Render category origin, classification, and state with text
  and icon/label support; never infer financial or status meaning from category
  color.
- Use versioned authenticated JSON resource routes and stable `{ data: ... }`
  responses matching financial accounts. Laravel documentation confirms Form
  Request authorization/validation, policy checks, API Resources, and feature
  tests fit this model. Vue documentation supports extracting stateful feature
  behavior into composables/stores; Element Plus supports promise-based
  confirmations and form-rule validation used by existing screens.

## Phase 1 Design Outputs

- Research decisions: [research.md](research.md)
- Data model: [data-model.md](data-model.md)
- API contract: [contracts/categories-api.yaml](contracts/categories-api.yaml)
- Setup and verification: [quickstart.md](quickstart.md)
- Agent context: [AGENTS.md](../../AGENTS.md) now points to this plan.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |
