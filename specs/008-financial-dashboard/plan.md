# Implementation Plan: Financial Dashboard

**Branch**: `008-financial-dashboard` | **Date**: 2026-09-17 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/008-financial-dashboard/spec.md`

**Branch Coordination**: `008-financial-dashboard` is checked out in specs,
`../zunera-backend`, and `../zunera-frontend`.

## Summary

Deliver read-only, owner-scoped financial dashboard. Backend derives every
section from authoritative records—never persisted dashboard totals—using
existing effective, removal, balance, transfer, archive, and recurrence rules.
Independent projections keep a failed chart/activity section from blocking
reliable sections. Frontend makes `/app` Dashboard landing page and composes
accessible responsive summary, account, activity, and data-led visualization
sections after backend contract, authorization, validation, and tests pass.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13.17+; JavaScript/Vue 3.5.40  
**Primary Dependencies**: Sanctum 4, Eloquent, Axios 1.19, Pinia 4, Element Plus
2.14.5, Tailwind 4.3, Vue Router 5.2; no chart package  
**Storage**: Existing relational records; integer BRL centavos; no dashboard
aggregate persistence  
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.61, Redocly  
**Target Platform**: Authenticated Zunera web application  
**Project Type**: Full-stack web feature  
**Performance Goals**: Usable dashboard sections within 2 seconds for up to
10,000 owned financial movements after opening or changing period  
**Constraints**: Protected read-only API; owner isolation; exact centavos;
America/Sao_Paulo business date; no new packages; source records are sole truth;
30-day future-only expected horizon  
**Scale/Scope**: Six independently retriable projections: summary, accounts,
expense distribution, evolution, recent activity, upcoming activity.

## Delivery Scope and Order

1. **Backend** — Contract, Sanctum authorization, Form Request period validation,
   API Resources, projection services, existing-rule reuse, measured indexes if
   needed, and feature/unit/performance tests in `../zunera-backend`.
2. **Frontend** — Dashboard route/home label, Axios service, focused Pinia store,
   feature components, accessible native data visualizations, i18n, and
   unit/Playwright tests in `../zunera-frontend`.

Frontend starts only after the backend contract, authorization, validation,
services, and tests of the story it consumes are implemented.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] Sanctum middleware, Form Request, API Resources, and no uploads protect
  every new API boundary.
- [x] Backend contract/authorization/validation/service/tests work precedes the
  frontend work that consumes it.
- [x] Exact feature branch is active in specs, backend, and frontend.
- [x] Read projection services own business rules; period DTO is justified;
  repository is not justified because scoped existing models/services suffice.
- [x] Frontend uses JavaScript `<script setup>` Composition API, Axios service,
  focused Pinia state, Element Plus controls, Tailwind tokens, and props/events.
- [x] Contract, backend unit/feature/performance, frontend unit/service/store,
  and isolated Playwright coverage are planned.
- [x] Ownership, no persistence, no secrets/uploads, accessibility, and excluded
  scope are explicit.
- [x] No constitution exception required.

## Project Structure

### Documentation (this feature)

```text
specs/008-financial-dashboard/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── checklists/requirements.md
└── contracts/financial-dashboard-api.yaml
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/{Data,Http,Services}/FinancialDashboard/
├── app/Services/RecurringTransactions/RecurringScheduleCalculator.php
├── routes/api.php
└── tests/{Feature,Unit}/FinancialDashboard/

../zunera-frontend/
├── src/components/dashboard/
├── src/views/dashboard/FinancialDashboardView.vue
├── src/services/dashboardService.js
├── src/stores/dashboard/dashboardStore.js
├── src/utils/dashboard/dashboardFormatters.js
├── src/{router/index.js,layouts/AppShell.vue,i18n/messages.js}
└── e2e/financial-dashboard.spec.js
```

**Structure Decision**: Six backend read-projection services isolate calculation
and failure concerns. `FinancialDashboardView` stays composition-only; cards
receive state/data as props and emit retry, navigation, period intent. No chart
dependency exists, so feature-local semantic SVG/data-table visualizations use
existing tokens and retain textual alternatives.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |
