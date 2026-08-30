# Tasks: User Authentication

**Input**: Design documents from `/specs/001-user-auth/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/auth-api.yaml`, `quickstart.md`

**Tests**: Required by SQR-005 and the project constitution. Write focused tests before each implementation and verify backend contracts before frontend consumption.

**Organization**: Tasks are grouped by user story. Within every story, backend contract, validation, services, and tests complete before frontend work begins.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it uses different files and has no incomplete dependency.
- **[Story]**: Maps the task to US1, US2, or US3.
- Every task names the exact file or files it changes or validates.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm coordinated branches and expose required environment settings without adding packages.

- [ ] T001 Verify branch `001-user-auth` is active in `.git/HEAD`, `../zunera-backend/.git/HEAD`, and `../zunera-frontend/.git/HEAD`
- [ ] T002 Add documented Sanctum stateful domains, frontend URL, credentialed CORS, 15-minute session lifetime, auth-mail queue, and production mail variables to `../zunera-backend/.env.example`
- [ ] T003 [P] Add the backend base URL and LGPD privacy/privacy-rights URL variables to `../zunera-frontend/.env.example`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish secure SPA sessions, shared validation, response resources, and database compatibility used by every story.

**⚠️ CRITICAL**: No user-story implementation begins until this phase passes.

- [ ] T004 Write failing stateful-cookie, CSRF, JSON unauthenticated-response, and secret-redaction tests in `../zunera-backend/tests/Feature/Auth/AuthenticationInfrastructureTest.php`
- [ ] T005 Enable Sanctum stateful API middleware, authenticated-session middleware, and JSON API error rendering in `../zunera-backend/bootstrap/app.php`
- [ ] T006 [P] Configure exact credentialed origins, secure session cookies, 15-minute idle lifetime, and authentication mail queue in `../zunera-backend/config/cors.php`, `../zunera-backend/config/session.php`, and `../zunera-backend/config/queue.php`
- [ ] T007 [P] Create the nullable-name migration required by email-only registration in `../zunera-backend/database/migrations/2026_08_30_000001_make_user_name_nullable.php`
- [ ] T008 [P] Implement canonical email normalization and the shared 15–64 character compromised-password rule in `../zunera-backend/app/Services/Authentication/EmailNormalizer.php` and `../zunera-backend/app/Services/Authentication/PasswordRules.php`
- [ ] T009 [P] Create allow-listed user/session API resources in `../zunera-backend/app/Http/Resources/Auth/UserResource.php` and `../zunera-backend/app/Http/Resources/Auth/AuthenticatedSessionResource.php`
- [ ] T010 Standardize generic authentication, validation, throttle, CSRF, and expired-session JSON outcomes in `../zunera-backend/app/Exceptions/AuthenticationException.php` and `../zunera-backend/bootstrap/app.php`
- [ ] T011 Run the foundational feature test and migration checks, then reconcile infrastructure assumptions in `specs/001-user-auth/contracts/auth-api.yaml`

**Checkpoint**: Stateful session and shared backend foundations are verified.

---

## Phase 3: User Story 1 - Create an Account (Priority: P1) 🎯 MVP

**Goal**: A visitor creates one account with normalized email and a compliant password, receives privacy-safe feedback, and enters an authenticated session.

**Independent Test**: Register with valid email/password data and verify one account, an authenticated session, protected access, duplicate/repeated-submit safety, validation feedback, and LGPD links.

### Backend Tests for User Story 1

- [ ] T012 [P] [US1] Write failing registration contract tests for success, normalization, duplicate email, validation, generic duplicate guidance, idempotent repeated submission, and authenticated response shape in `../zunera-backend/tests/Feature/Auth/RegistrationTest.php`
- [ ] T013 [P] [US1] Write failing unit tests for email normalization, 15–64 character policy, printable spaces, confirmation, common-password rejection, and compromised-verifier outcomes in `../zunera-backend/tests/Unit/Authentication/EmailNormalizerTest.php` and `../zunera-backend/tests/Unit/Authentication/PasswordRulesTest.php`

### Backend Implementation for User Story 1

- [ ] T014 [US1] Implement normalized registration validation and immutable payload mapping in `../zunera-backend/app/Http/Requests/Auth/RegisterRequest.php` and `../zunera-backend/app/Data/Authentication/RegisterData.php`
- [ ] T015 [US1] Implement transactional account creation, unique-conflict handling, login, session regeneration, and absolute-expiry metadata in `../zunera-backend/app/Services/Authentication/AuthenticationService.php`
- [ ] T016 [US1] Expose `POST /api/v1/auth/register` with the verified resource response in `../zunera-backend/app/Http/Controllers/Api/V1/AuthController.php` and `../zunera-backend/routes/api.php`
- [ ] T017 [US1] Run `RegistrationTest` and authentication unit tests, then verify status codes and schemas against `specs/001-user-auth/contracts/auth-api.yaml`

### Frontend Tests for User Story 1

- [ ] T018 [P] [US1] Write failing CSRF, registration response, and validation-error mapping tests in `../zunera-frontend/src/services/__tests__/authService.registration.spec.js`
- [ ] T019 [P] [US1] Write failing registration/bootstrap/session-state tests in `../zunera-frontend/src/stores/auth/__tests__/sessionStore.registration.spec.js`
- [ ] T020 [P] [US1] Write failing top-label, password-guidance, validation, loading-lock, duplicate-feedback, and emitted-payload tests in `../zunera-frontend/src/components/auth/__tests__/RegisterForm.spec.js`

### Frontend Implementation for User Story 1

- [ ] T021 [US1] Bootstrap Tailwind, Element Plus, semantic Zunera tokens, and the root router outlet in `../zunera-frontend/vite.config.js`, `../zunera-frontend/src/main.js`, `../zunera-frontend/src/assets/main.css`, `../zunera-frontend/src/assets/theme.css`, and `../zunera-frontend/src/App.vue`
- [ ] T022 [US1] Implement credentialed Axios, one-time CSRF refresh/retry, safe error normalization, and registration transport in `../zunera-frontend/src/services/httpClient.js` and `../zunera-frontend/src/services/authService.js`
- [ ] T023 [US1] Implement setup-style shared session bootstrap and registration actions without persisting credentials in `../zunera-frontend/src/stores/auth/sessionStore.js`
- [ ] T024 [P] [US1] Create the centered authentication layout and reusable LGPD links in `../zunera-frontend/src/layouts/AuthLayout.vue` and `../zunera-frontend/src/components/auth/PrivacyNoticeLinks.vue`
- [ ] T025 [US1] Create `RegisterForm` and shared password guidance using `<ElForm label-position="top">`, `<ElFormItem>`, durable alerts, and one loading state in `../zunera-frontend/src/components/auth/RegisterForm.vue` and `../zunera-frontend/src/components/auth/PasswordRequirements.vue`
- [ ] T026 [US1] Create registration orchestration and a minimal protected landing target in `../zunera-frontend/src/views/auth/RegisterView.vue` and `../zunera-frontend/src/views/ProtectedHomeView.vue`
- [ ] T027 [US1] Add registration, protected landing, guest-only, and intended-destination routing in `../zunera-frontend/src/router/index.js` and `../zunera-frontend/src/router/authGuard.js`
- [ ] T028 [US1] Add account-creation Playwright coverage for success, duplicate/repeated submit, validation, protected access, 320px width, keyboard use, and LGPD links in `../zunera-frontend/e2e/authentication-registration.spec.js`
- [ ] T029 [US1] Run registration Vitest, Playwright, and production build checks and record verified commands in `specs/001-user-auth/quickstart.md`

**Checkpoint**: User Story 1 is independently usable as the MVP.

---

## Phase 4: User Story 2 - Sign In and Sign Out (Priority: P1)

**Goal**: A registered user signs in, accesses protected content, receives progressive-delay and expiry behavior, continues before idle expiry, and signs out the current session.

**Independent Test**: Sign in with valid and invalid credentials, verify throttling and protected-route denial, continue at the warning, enforce both expiry limits, then sign out and confirm the session cannot be reused.

### Backend Tests for User Story 2

- [ ] T030 [P] [US2] Write failing login, session-read, continuation, logout, generic-credential, session-regeneration, and current-session invalidation tests in `../zunera-backend/tests/Feature/Auth/LoginSessionLogoutTest.php`
- [ ] T031 [P] [US2] Write failing 15-minute idle, one-minute warning metadata, meaningful-activity continuation, polling exclusion, and eight-hour absolute-expiry tests in `../zunera-backend/tests/Feature/Auth/SessionLifetimeTest.php`
- [ ] T032 [P] [US2] Write failing five-attempt window, 1/2/4/8/15-minute delay, `Retry-After`, unknown-account equivalence, recovery availability, and success-clear tests in `../zunera-backend/tests/Unit/Authentication/ProgressiveLoginLimiterTest.php`

### Backend Implementation for User Story 2

- [ ] T033 [US2] Implement normalized login validation and DTO mapping in `../zunera-backend/app/Http/Requests/Auth/LoginRequest.php` and `../zunera-backend/app/Data/Authentication/LoginData.php`
- [ ] T034 [US2] Implement HMAC-keyed account/IP throttling and retry metadata in `../zunera-backend/app/Services/Authentication/ProgressiveLoginLimiter.php` and `../zunera-backend/app/Exceptions/LoginThrottledException.php`
- [ ] T035 [US2] Implement login, generic failure, session read/continue, current-session logout, idle tracking, and absolute expiry in `../zunera-backend/app/Services/Authentication/AuthenticationService.php` and `../zunera-backend/app/Http/Middleware/EnforceSessionLifetime.php`
- [ ] T036 [US2] Expose login, session read/continue, and logout routes with `auth:sanctum` protection in `../zunera-backend/app/Http/Controllers/Api/V1/AuthController.php` and `../zunera-backend/routes/api.php`
- [ ] T037 [US2] Run login/session/limiter tests and verify response bodies, `401`, `429`, `Retry-After`, and expiry timestamps against `specs/001-user-auth/contracts/auth-api.yaml`

### Frontend Tests for User Story 2

- [ ] T038 [P] [US2] Write failing login, logout, session, continuation, invalid-credential, throttle, `419`, and `401` transport tests in `../zunera-frontend/src/services/__tests__/authService.session.spec.js`
- [ ] T039 [P] [US2] Write failing bootstrap, login, logout, invalidation, and authoritative-expiry state tests in `../zunera-frontend/src/stores/auth/__tests__/sessionStore.session.spec.js`
- [ ] T040 [P] [US2] Write failing top-label, retained-email, cleared-password, inline-error, alert, and loading-lock tests in `../zunera-frontend/src/components/auth/__tests__/SignInForm.spec.js`
- [ ] T041 [P] [US2] Write failing one-minute warning, focus trap, keyboard, continue, sign-out, and expiration tests in `../zunera-frontend/src/components/auth/__tests__/SessionExpiryDialog.spec.js`

### Frontend Implementation for User Story 2

- [ ] T042 [US2] Add login, logout, session bootstrap, continuation, throttle, and invalidation methods/actions in `../zunera-frontend/src/services/authService.js` and `../zunera-frontend/src/stores/auth/sessionStore.js`
- [ ] T043 [US2] Create sign-in UI with `<ElForm label-position="top">`, durable generic feedback, retained email, cleared password, and loading lock in `../zunera-frontend/src/components/auth/SignInForm.vue` and `../zunera-frontend/src/views/auth/SignInView.vue`
- [ ] T044 [US2] Implement server-authoritative idle/absolute timers and accessible warning behavior in `../zunera-frontend/src/composables/useSessionExpiry.js` and `../zunera-frontend/src/components/auth/SessionExpiryDialog.vue`
- [ ] T045 [US2] Enforce protected/guest routes, intended internal redirects, bootstrap-before-render, and global expiry handling in `../zunera-frontend/src/router/authGuard.js`, `../zunera-frontend/src/router/index.js`, and `../zunera-frontend/src/App.vue`
- [ ] T046 [US2] Add sign-in, invalid credential, protected route, logout, and session-expiry Playwright journeys in `../zunera-frontend/e2e/authentication-session.spec.js`
- [ ] T047 [P] [US2] Add progressive-delay, recovery-availability, keyboard, alert-announcement, 200%-zoom, and 320px Playwright coverage in `../zunera-frontend/e2e/authentication-security-accessibility.spec.js`
- [ ] T048 [US2] Run session Vitest, Playwright, and production build checks and record verified commands in `specs/001-user-auth/quickstart.md`

**Checkpoint**: User Stories 1 and 2 work independently and together.

---

## Phase 5: User Story 3 - Recover Account Access (Priority: P2)

**Goal**: A visitor receives neutral recovery feedback, uses only the newest valid 60-minute instruction once, resets the password, ends all sessions, and receives a security notice.

**Independent Test**: Request recovery for known and unknown addresses, validate identical outcomes and the three-email limit, reset through the newest token, reject expired/used/superseded tokens, and confirm prior password/session invalidation and notification content.

### Backend Tests for User Story 3

- [ ] T049 [P] [US3] Write failing neutral-response, equivalent-timing-path, three-emails-per-hour, queued-delivery, unknown-email, and retry tests in `../zunera-backend/tests/Feature/Auth/PasswordRecoveryRequestTest.php`
- [ ] T050 [P] [US3] Write failing valid, expired, used, superseded, malformed, validation-preserved, concurrent one-time reset, old-password denial, remember-token rotation, and all-session deletion tests in `../zunera-backend/tests/Feature/Auth/PasswordResetTest.php`
- [ ] T051 [P] [US3] Write failing encrypted-queue, after-commit, superseded-send suppression, five-minute metric, and safe security-notice-content tests in `../zunera-backend/tests/Unit/Authentication/AuthenticationNotificationTest.php`

### Backend Implementation for User Story 3

- [ ] T052 [US3] Implement normalized recovery/reset validation and DTOs in `../zunera-backend/app/Http/Requests/Auth/RequestPasswordRecoveryRequest.php`, `../zunera-backend/app/Http/Requests/Auth/ResetPasswordRequest.php`, `../zunera-backend/app/Data/Authentication/RecoveryRequestData.php`, and `../zunera-backend/app/Data/Authentication/ResetPasswordData.php`
- [ ] T053 [US3] Implement neutral queued recovery handling and the HMAC-keyed three-send rolling-hour limit in `../zunera-backend/app/Services/Authentication/PasswordRecoveryService.php` and `../zunera-backend/app/Services/Authentication/RecoveryEmailLimiter.php`
- [ ] T054 [US3] Implement transactional locked-token validation, password update, token consumption, remember-token rotation, and database-session deletion in `../zunera-backend/app/Services/Authentication/PasswordResetService.php`
- [ ] T055 [US3] Implement encrypted queued reset instructions, current-token recheck, and password-change security notices after commit in `../zunera-backend/app/Notifications/Auth/ResetPasswordNotification.php` and `../zunera-backend/app/Notifications/Auth/PasswordChangedNotification.php`
- [ ] T056 [US3] Expose neutral recovery and reset endpoints with verified status/error shapes in `../zunera-backend/app/Http/Controllers/Api/V1/AuthController.php` and `../zunera-backend/routes/api.php`
- [ ] T057 [US3] Run recovery/reset/notification tests and verify request, success, and invalid-link schemas against `specs/001-user-auth/contracts/auth-api.yaml`

### Frontend Tests for User Story 3

- [ ] T058 [P] [US3] Write failing neutral recovery, reset success, expired/used/superseded mapping, and token-redaction transport tests in `../zunera-frontend/src/services/__tests__/authService.recovery.spec.js`
- [ ] T059 [P] [US3] Write failing top-label, neutral confirmation, retry, loading-lock, password-guidance, invalid-link, and local-token handling tests in `../zunera-frontend/src/components/auth/__tests__/RecoveryAndResetForms.spec.js`

### Frontend Implementation for User Story 3

- [ ] T060 [US3] Add recovery and reset transports with safe error normalization and no token persistence in `../zunera-frontend/src/services/authService.js`
- [ ] T061 [US3] Create recovery request/confirmation UI using `<ElForm label-position="top">`, durable neutral feedback, troubleshooting, retry, sign-in path, and LGPD links in `../zunera-frontend/src/components/auth/RecoveryRequestForm.vue`, `../zunera-frontend/src/components/auth/RecoveryConfirmation.vue`, and `../zunera-frontend/src/views/auth/ForgotPasswordView.vue`
- [ ] T062 [US3] Create reset UI using `<ElForm label-position="top">`, shared password guidance, invalid-link states, and a sign-in path in `../zunera-frontend/src/components/auth/ResetPasswordForm.vue`, `../zunera-frontend/src/components/auth/RecoveryLinkState.vue`, and `../zunera-frontend/src/views/auth/ResetPasswordView.vue`
- [ ] T063 [US3] Add recovery/reset routes while keeping email and reset token route-local and out of Pinia/storage in `../zunera-frontend/src/router/index.js`
- [ ] T064 [US3] Add known/unknown recovery, neutral confirmation, newest-token, expired/superseded/used link, reset, old-password, and session-invalidation Playwright journeys in `../zunera-frontend/e2e/authentication-recovery.spec.js`
- [ ] T065 [US3] Run recovery/reset Vitest, Playwright, and production build checks and record verified commands in `specs/001-user-auth/quickstart.md`

**Checkpoint**: All three authentication stories are independently functional.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Complete security operations, retention, contract checks, accessibility evidence, and full regression verification.

- [ ] T066 Add secret-free structured auth events and queue/delivery/throttle/expiry metrics in `../zunera-backend/app/Services/Authentication/AuthenticationAudit.php` and `../zunera-backend/config/logging.php`
- [ ] T067 Add expired recovery-token, session, cache-key, failed-job, and security-log retention scheduling in `../zunera-backend/routes/console.php` and document retention variables in `../zunera-backend/.env.example`
- [ ] T068 [P] Validate and reconcile every implemented operation/schema in `specs/001-user-auth/contracts/auth-api.yaml`
- [ ] T069 [P] Complete keyboard, focus, alert, contrast, light/dark/system theme, 200% zoom, and 320px regression coverage in `../zunera-frontend/e2e/authentication-security-accessibility.spec.js`
- [ ] T070 Run the full backend PHPUnit suite and Pint, then record results in `specs/001-user-auth/quickstart.md`
- [ ] T071 Run the full frontend Vitest suite, production build, and isolated Playwright suite, then record results in `specs/001-user-auth/quickstart.md`
- [ ] T072 Execute every manual smoke scenario and confirm the 2-second auth, five-minute recovery-mail, session-expiry, neutral-response, and notification outcomes in `specs/001-user-auth/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1** has no dependencies.
- **Phase 2** depends on Phase 1 and blocks all stories.
- **US1** starts after Phase 2; its backend T012–T017 must pass before its frontend T018–T029.
- **US2** starts after Phase 2; its backend T030–T037 must pass before its frontend T038–T048.
- **US3** starts after Phase 2; its backend T049–T057 must pass before its frontend T058–T065.
- **Phase 6** follows all stories selected for release.

### User Story Dependencies

- **US1 (P1)**: No story dependency after the foundation; it is the MVP.
- **US2 (P1)**: No domain dependency on US1, but it reuses the shared session resource and frontend bootstrap.
- **US3 (P2)**: No domain dependency on US1/US2; reset-session invalidation reuses the shared session store and database sessions.
- For one-developer delivery, follow US1 → US2 → US3. With multiple owners, backend work may proceed in parallel after Phase 2, but each story's frontend waits for that story's verified backend.

### Within Each Story

1. Write contract/feature/unit tests and confirm they fail for the missing behavior.
2. Implement Requests/DTOs before services, services before controller routes.
3. Run backend tests and reconcile the OpenAPI contract.
4. Write frontend tests, then implement services/store/components/views.
5. Run frontend unit, build, and isolated Playwright checks.

## Parallel Execution Examples

### User Story 1

```text
Parallel backend: T012 RegistrationTest | T013 normalizer/password unit tests
Parallel frontend after T017: T018 auth service tests | T019 store tests | T020 form tests
Parallel UI after T023: T024 AuthLayout/privacy links while T025 registration form is prepared
```

### User Story 2

```text
Parallel backend: T030 login/logout tests | T031 lifetime tests | T032 limiter tests
Parallel frontend after T037: T038 service tests | T039 store tests | T040 form tests | T041 dialog tests
Parallel E2E after UI integration: T046 session journeys | T047 security/accessibility journeys
```

### User Story 3

```text
Parallel backend: T049 recovery tests | T050 reset tests | T051 notification tests
Parallel frontend after T057: T058 service tests | T059 component tests
```

## Implementation Strategy

### MVP First

1. Complete Setup and Foundational phases.
2. Complete US1 backend T012–T017.
3. Complete US1 frontend T018–T029.
4. Stop and validate account creation independently before adding session and recovery journeys.

### Incremental Delivery

1. Deliver US1 account creation as the first usable increment.
2. Add US2 sign-in/sign-out and server-authoritative session lifecycle.
3. Add US3 recovery/reset and security notification flows.
4. Finish cross-cutting observability, retention, accessibility, and full verification.

### Team Parallelism

After Phase 2, separate owners may implement the three backend story slices concurrently. Frontend owners start only when the corresponding backend contract, validation, authorization, and tests are verified. Avoid parallel edits to shared `AuthController.php`, `routes/api.php`, `authService.js`, or `sessionStore.js` without explicit sequencing.

## Notes

- No new Composer or npm packages are required.
- All Vue forms use Element Plus `<ElForm label-position="top">` and `<ElFormItem>`.
- Passwords, raw recovery tokens, cookies, session IDs, and submitted emails never enter logs or persistent frontend storage.
- Commit after each task or coherent task group; do not mix unrelated refactors.
