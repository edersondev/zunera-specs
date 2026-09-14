# Implementation Plan: Account Transfers

**Branch**: 005-account-transfers | **Date**: 2026-09-13 | **Spec**:
[spec.md](spec.md)
**Input**: Feature specification from specs/005-account-transfers/spec.md

**Branch Coordination**: 005-account-transfers is checked out in zunera-specs,
../zunera-backend, and ../zunera-frontend.

**Note**: Planning ends after Phase 1 design. Task generation belongs to
/speckit-tasks.

## Summary

Deliver owned BRL transfers between two financial accounts without creating or
destroying money. Backend first defines protected lifecycle, atomic two-account
reconciliation, no-overdraft policy, recoverable removal, history integration,
contracts, and tests. Frontend then delivers accessible Transfers workspace and
labelled transfers beside income/expense in existing financial history.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13.17+; JavaScript with Vue 3.5.40  
**Primary Dependencies**: Laravel Sanctum 4, Form Requests, API Resources,
Eloquent; Axios 1.19, Pinia 4.0.2, Vue Router 5.2, Element Plus 2.14.5,
Tailwind CSS 4.3, Vite 8.1  
**Storage**: Existing relational persistence; integer-centavo amounts/balances,
transfer records, and retained idempotency records.  
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.62, Redocly
OpenAPI lint  
**Target Platform**: Authenticated Zunera web application  
**Project Type**: Full-stack web feature across specs, backend API, frontend SPA  
**Performance Goals**: First 50-item batch from 5,000 transfers in 2 seconds,
verified by an elapsed-time assertion under a documented seeded test environment;
filter/search/save match authenticated history behavior.  
**Constraints**: No packages; protected routes; service-owned rules; exact BRL
centavos; atomic two-account outcome; effective transfer never overdraws source;
archives historical only; transfer never pollutes income/expense/report totals.  
**Scale/Scope**: At least 5,000 transfers/user; BRL; individual transfers only;
no peer, bank, PIX, recurring, investment, credit-card, conversion flows.

## Delivery Scope and Order

1. **Backend** — Transfer/mixed-history contracts, authorization, validation,
   services, persistence, two-account reconciliation, reporting classification,
   backend tests in ../zunera-backend.
2. **Frontend** — Transfer views, service, Pinia state, Element Plus controls,
   existing-history rendering, frontend tests in ../zunera-frontend.

Frontend implementation MUST start only after backend contract, authorization,
validation, and tests complete.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] API boundaries use authentication/session middleware, Form Requests, API
      Resources, and no uploads.
- [x] Backend contract, authorization, validation, tests precede frontend work.
- [x] Branch names match across all three repositories.
- [x] Transfer rules live in focused services; multi-field payloads justify DTOs;
      direct owned associations do not justify repository abstraction.
- [x] Frontend uses Composition API, feature services, Pinia only for shared
      state, and Element Plus controls.
- [x] Contract, backend unit/feature, frontend unit, isolated Playwright
      coverage planned for every changed behavior.
- [x] Ownership/privacy, idempotency, no uploads/secrets, scope limits recorded.
- [x] No constitution exception required.

## Project Structure

### Documentation (this feature)

    specs/005-account-transfers/
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    ├── checklists/requirements.md
    ├── contracts/transfers-api.yaml
    ├── contracts/financial-history-api.yaml
    └── tasks.md

### Source Code (sibling repositories)

    ../zunera-backend/
    ├── app/
    │   ├── Data/Transfers/
    │   ├── Enums/Transfers/
    │   ├── Exceptions/Transfers/
    │   ├── Http/
    │   │   ├── Controllers/Api/V1/TransferController.php
    │   │   ├── Controllers/Api/V1/FinancialHistoryController.php
    │   │   ├── Requests/Transfers/
    │   │   └── Resources/Transfers/
    │   ├── Models/Transfer.php
    │   ├── Models/TransferMutationRequest.php
    │   └── Services/Transfers/
    ├── database/factories/TransferFactory.php
    ├── database/migrations/
    ├── routes/api.php
    └── tests/Feature/Transfers, tests/Feature/FinancialHistory,
        tests/Unit/Transfers

    ../zunera-frontend/
    ├── e2e/transfers.spec.js
    ├── src/components/transfers/
    ├── src/router/index.js
    ├── src/services/transferService.js
    ├── src/stores/transfers/transferStore.js
    ├── src/utils/transfers/
    └── src/views/transfers/

**Structure Decision**: Follow existing financial vertical slices. Backend groups
transfer services, requests, resources, and tests by domain; repository not
needed. Frontend separates transport service, shared store, pure formatters,
feature components, views. Existing history changes only to render discriminated
transfer entry. FinancialHistoryController solely owns `GET /financial-history`
for that mixed projection; the existing transaction-only `GET /transactions`
route remains owned by feature 004's transaction controller.

## Phase 0 Research Decisions

- Exact BRL centavos, materialized balances, delta reconciliation, deterministic
  two-account locks, idempotency, lifecycle, archive behavior, mixed-history
  contract, and frontend boundaries: [research.md](research.md).

## Phase 1 Design Outputs

- Data model: [data-model.md](data-model.md)
- Transfer API: [contracts/transfers-api.yaml](contracts/transfers-api.yaml)
- Canonical mixed financial-history read projection:
  [contracts/financial-history-api.yaml](contracts/financial-history-api.yaml)
- Setup/verification: [quickstart.md](quickstart.md)
- Agent context: [AGENTS.md](../../AGENTS.md) points to this plan.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |
