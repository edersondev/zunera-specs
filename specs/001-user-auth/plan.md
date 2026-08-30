# Implementation Plan: User Authentication

**Branch**: `001-user-auth` | **Date**: 2026-08-30 | **Spec**: [spec.md](spec.md)

**Branch Coordination**: The same branch exists in `zunera-specs`,
`../zunera-backend`, and `../zunera-frontend`.

## Summary

Deliver email/password registration, sign-in, sign-out, recovery, and secure
reset for Brazilian Zunera users. Backend establishes a versioned, cookie-based
Sanctum session contract, validation, throttling, expiry, queued notifications,
and tests. Frontend consumes that contract through a Composition API service
and Pinia session store, with Element Plus forms using `label-position="top"`
and the Zunera Design Foundation.

## Technical Context

**Language/Version**: PHP 8.3 / Laravel 13.26.1; JavaScript ES modules / Vue 3.5.41
**Primary Dependencies**: Sanctum 4.3.3, PHPUnit 12.5.33, Vue Router 5.2.0, Pinia 4.0.3, Axios 1.19.0, Element Plus 2.14.5, Vitest 4.1.11, Playwright 1.62.1
**Storage**: Existing MySQL-compatible users, database sessions, password-reset-token, cache, queue, and SMTP services
**Testing**: PHPUnit feature/unit, scripted production-like API percentile checks, auth-mail delivery-event acceptance checks, Vitest component/service/store, Playwright Chromium/Firefox/WebKit, Pint
**Target Platform**: Credentialed browser SPA and Laravel JSON API
**Project Type**: Full-stack web feature
**Performance Goals**: p95 normal auth outcomes within 2 seconds across 100 production-like actions; at least 95 of 100 valid-account recovery emails reach a provider delivery event within 5 minutes
**Constraints**: No password/token leakage; fail-closed compromised-password checks; 15-minute idle and 8-hour absolute limits; stable recovery-link error codes; neutral recovery; LGPD; no new packages; the complete backend contract/tests precede all frontend changes
**Scale/Scope**: Authentication surfaces and API only; use existing deployment capacity with auth-mail, throttle, and expiry metrics

## Delivery Scope and Order

1. **Backend** — Add nullable `users.name` migration, Form Requests, DTOs,
   authentication/recovery services, thin API controllers, Resources,
   Sanctum/CORS middleware, session-expiry middleware, cache-backed throttles,
   queued notifications, versioned routes, API contract, and PHPUnit coverage.
   Reset invalidates every session; ordinary logout only invalidates the current.
2. **Frontend** — Add Axios/CSRF client, auth service, setup-style Pinia session
   store, route guards, auth layout/views/forms, recovery-link states, session
   expiry dialog, semantic theme/bootstrap, and Vitest/Playwright coverage.
   Passwords and reset secrets stay local to forms and are never persisted.

No frontend file may be changed until registration, session, recovery/reset,
cross-cutting backend behavior, the full API contract, authorization, validation,
and backend tests are complete and the backend gate is recorded as passing.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- [x] API boundaries identify Form Requests, middleware, API Resources; no uploads are in scope.
- [x] Backend scope and API contract precede frontend; frontend consumes `contracts/auth-api.yaml`.
- [x] Feature branch matches in all three repositories.
- [x] Business rules live in services; DTOs cover multi-field payloads; no repository is needed for simple auth tables.
- [x] Frontend uses Composition API, API services, Pinia shared state, and Element Plus controls/forms.
- [x] PHPUnit, Vitest, contract examples, and isolated Playwright journeys cover changed/security behavior.
- [x] Security, environment configuration, LGPD, and scope are recorded in research and quickstart.
- [x] No constitution exception.

## Project Structure

### Documentation (this feature)

```text
specs/001-user-auth/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/auth-api.yaml
└── tasks.md                 # created later by /speckit-tasks
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/Http/Controllers/Api/V1/AuthController.php
├── app/Http/Requests/Auth/
├── app/Http/Resources/Auth/
├── app/Services/Authentication/
├── app/Notifications/Auth/
├── app/Http/Middleware/EnforceSessionLifetime.php
├── app/Console/Commands/VerifyAuthMailDelivery.php
├── scripts/auth-performance-smoke.sh
├── docs/auth-mail-operations.md
├── database/migrations/*_make_user_name_nullable.php
├── routes/api.php
└── tests/Feature/Auth/ and tests/Unit/Authentication/

../zunera-frontend/
├── src/services/{httpClient.js,authService.js}
├── src/stores/auth/sessionStore.js
├── src/router/{index.js,authGuard.js}
├── src/views/auth/ and src/components/auth/
├── src/layouts/AuthLayout.vue
├── src/composables/useSessionExpiry.js
├── src/assets/{main.css,theme.css}
├── src/__tests__/auth/
└── e2e/authentication*.spec.js
```

**Structure Decision**: Backend work in `../zunera-backend` precedes frontend
work in `../zunera-frontend`; each layer owns validation, tests, and framework
conventions while sharing the checked-in API contract.

## Complexity Tracking

No violations. Custom session-lifetime middleware and services implement
security rules absent from the starter skeleton.
