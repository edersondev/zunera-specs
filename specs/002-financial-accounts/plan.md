# Implementation Plan: Financial Accounts

**Branch**: `002-financial-accounts` | **Date**: 2026-09-02 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/002-financial-accounts/spec.md`

**Branch Coordination**: Current branch is `002-financial-accounts` in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`.

**Note**: This plan stops at Phase 2 planning. Task generation remains the
responsibility of `/speckit-tasks`.

## Summary

Build authenticated financial account management for Zunera with backend-first
delivery. The backend defines protected account contracts, user ownership,
validation, lifecycle actions, BRL centavo balance handling, initial-balance
locking rules, normalized active-name uniqueness, and tests before the frontend
consumes the API in the authenticated Vue app shell. Account name is the user's
own label and remains separate from optional financial institution text.

## Technical Context

**Language/Version**: Backend PHP 8.3 with Laravel 13.26.1; frontend JavaScript
with Vue 3.5.41 and Vite 8.2.2  
**Primary Dependencies**: Laravel Sanctum 4.3.3, Laravel API Resources/Form
Requests, Pinia 4.0.3, Axios 1.19.0, Element Plus 2.14.5, Vue Router 5.2.0,
Maska 3.2.0  
**Storage**: Existing Laravel application persistence; feature data lives in the
backend application and is user-owned  
**Testing**: PHPUnit 12.5.33, Laravel Pint 1.30.5, Vitest 4.1.11, Playwright
1.62.1, Redocly OpenAPI lint  
**Target Platform**: Authenticated web application with Laravel JSON API backend
and Vue SPA frontend  
**Project Type**: Full-stack web feature across specs, backend API, and frontend
SPA repositories  
**Performance Goals**: No extra feature-specific load target beyond normal
interactive account-management validation; no bulk or reporting workload is in
scope  
**Constraints**: No new packages without approval; protected routes only; thin
controllers; service-owned business rules; DTOs for multi-field inputs; no
permanent deletion; BRL integer-centavo precision; per-account values capped at
+/-999,999,999,999 centavos; frontend follows Zunera Design Foundation, app
shell, navigation, and shared component rules  
**Scale/Scope**: Personal financial accounts owned by one user. No shared
accounts, bank integrations, imports, transactions, transfers, reconciliation,
statements, or movement-based balance calculation in this feature.

## Delivery Scope and Order

1. **Backend** - Laravel API contracts, authorization, validation, services,
   persistence, balance rules, lifecycle rules, ownership boundaries, and
   backend tests in `../zunera-backend`.
2. **Frontend** - Vue routes, account service, Pinia state, Element Plus UI,
   theme-aware responsive screens, and frontend tests in `../zunera-frontend`.

Frontend implementation MUST start only after the relevant backend API contract,
authorization, validation, and tests are complete.

## Constitution Check

*GATE: Passed before Phase 0 research. Re-check after Phase 1 design: Passed.*

- [x] API boundaries identify Form Requests, middleware, API Resources, and any
      upload validation/sanitization needed. No uploads are in scope.
- [x] Backend scope and API contract are complete before frontend work; frontend
      scope consumes that contract from `../zunera-frontend`.
- [x] Feature branch name matches in specs, backend, and frontend repositories:
      `002-financial-accounts`.
- [x] Business rules have service ownership; DTO use is justified by multi-field
      create/update payloads. Repository use is not planned because queries are
      straightforward and user-scoped.
- [x] Frontend design uses Composition API, services for API access, Pinia only
      for shared state, and Element Plus for standard interface controls.
- [x] Test and API-contract coverage covers changed behavior; critical UI-to-API
      journeys include isolated Playwright end-to-end coverage.
- [x] Security, environment configuration, and scope constraints are recorded.
- [x] No exception to the constitution is required.

## Project Structure

### Documentation (this feature)

```text
specs/002-financial-accounts/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/
│   └── requirements.md
├── contracts/
│   └── financial-accounts-api.yaml
└── tasks.md
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/
│   ├── Data/FinancialAccounts/
│   ├── Enums/FinancialAccounts/
│   ├── Exceptions/FinancialAccounts/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/FinancialAccounts/
│   │   └── Resources/FinancialAccounts/
│   ├── Models/
│   └── Services/FinancialAccounts/
├── database/
│   ├── factories/
│   └── migrations/
├── routes/api.php
└── tests/
    ├── Feature/FinancialAccounts/
    └── Unit/FinancialAccounts/

../zunera-frontend/
├── e2e/
├── src/
│   ├── components/financial-accounts/
│   ├── composables/financial-accounts/
│   ├── router/
│   ├── services/
│   ├── stores/financial-accounts/
│   ├── utils/financial-accounts/
│   └── views/financial-accounts/
└── tests through colocated `__tests__/` folders
```

**Structure Decision**: Backend work in `../zunera-backend` precedes frontend
work in `../zunera-frontend`. Keep financial-account domain files grouped by
feature while preserving existing Laravel and Vue project conventions.

## Phase 0 Research Decisions

- Use authenticated versioned JSON API routes protected by existing Sanctum and
  session lifetime middleware.
- Put financial-account business rules in a focused service with DTOs for
  create/update payloads and enums for account type/status.
- Store and exchange BRL money as signed integer centavos to preserve exact
  cent-level precision.
- Keep `current_balance_centavos` equal to `initial_balance_centavos` until
  future financial movement features define balance changes.
- Use `active` and `archived` lifecycle states only; no permanent deletion.
- Allow editing initial balance only while no financial movements exist.
- Enforce normalized active-name uniqueness per user; archived duplicates are
  allowed until restore or rename would create an active conflict.
- Treat account name as the user-defined label and financial institution as
  separate optional user-entered text.
- Use predefined accessible color and icon choices with defaults.
- Build frontend with the authenticated app shell, route-level views, API
  service, setup-style Pinia store, Element Plus controls, and Zunera theme
  tokens.

## Phase 1 Design Outputs

- Data model: [data-model.md](data-model.md)
- API contract: [contracts/financial-accounts-api.yaml](contracts/financial-accounts-api.yaml)
- Quickstart and verification guide: [quickstart.md](quickstart.md)
- Agent context: [AGENTS.md](../../AGENTS.md) already points to this plan file
  between the SPECKIT markers.

## Complexity Tracking

No constitution violations.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |
