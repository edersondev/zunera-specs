# Tasks: User Authentication

**Input**: Design artifacts from `/specs/001-user-auth/`
**Tests**: Required by SQR-005 and the constitution.
**Delivery gate**: Complete T001–T046 before changing any frontend file.

## Format

- `[P]` means different files and no incomplete dependency.
- `[US1]`, `[US2]`, and `[US3]` map work to the specification stories.
- Every task includes an exact file path.

## Phase 1: Backend Setup

**Purpose**: Coordinate branches and document backend environment settings.

- [ ] T001 Verify branch `001-user-auth` in `.git/HEAD`, `../zunera-backend/.git/HEAD`, and `../zunera-frontend/.git/HEAD`
- [ ] T002 Add Sanctum domains, frontend URL, credentialed CORS, 15-minute session lifetime, auth-mail queue/provider, delivery-event, and retention variables to `../zunera-backend/.env.example`

---

## Phase 2: Backend Foundation

**Purpose**: Build shared secure session, validation, response, and persistence foundations.

- [ ] T003 Write failing stateful-cookie, CSRF, JSON unauthenticated-response, and secret-redaction tests in `../zunera-backend/tests/Feature/Auth/AuthenticationInfrastructureTest.php`
- [ ] T004 Enable Sanctum stateful API middleware, authenticated-session middleware, and JSON API error rendering in `../zunera-backend/bootstrap/app.php`
- [ ] T005 [P] Configure exact credentialed origins, secure session cookies, 15-minute idle lifetime, and auth-mail queue in `../zunera-backend/config/cors.php`, `../zunera-backend/config/session.php`, and `../zunera-backend/config/queue.php`
- [ ] T006 [P] Create the nullable-name migration for email-only registration in `../zunera-backend/database/migrations/2026_08_30_000001_make_user_name_nullable.php`
- [ ] T007 [P] Implement email normalization and 15–64 character password validation with local common-password blocking and fail-closed remote-verifier handling in `../zunera-backend/app/Services/Authentication/EmailNormalizer.php` and `../zunera-backend/app/Services/Authentication/PasswordRules.php`
- [ ] T008 [P] Create allow-listed auth resources that serialize only user `id`/`email` and session expiry timestamps in `../zunera-backend/app/Http/Resources/Auth/UserResource.php` and `../zunera-backend/app/Http/Resources/Auth/AuthenticatedSessionResource.php`
- [ ] T009 Standardize validation, authentication, throttle, CSRF, session-expiry, `password_safety_unavailable`, `recovery_link_expired`, and `recovery_link_invalid` JSON errors in `../zunera-backend/app/Exceptions/AuthenticationException.php` and `../zunera-backend/bootstrap/app.php`
- [ ] T010 Run infrastructure tests and migration checks, then reconcile foundation responses in `specs/001-user-auth/contracts/auth-api.yaml`

**Checkpoint**: Backend foundation passes before story work.

---

## Phase 3: Backend User Story 1 - Create an Account (Priority: P1)

**Goal**: Create one normalized email-only account and authenticated session.
**Independent Test**: Verify success, validation, duplicate/repeated submission, fail-closed password safety, privacy-safe feedback, protected access, and absence of `name` in JSON.

### Tests

- [ ] T011 [P] [US1] Write failing registration contract tests for success, normalization, duplicate email, validation, repeated submission, authenticated response, `name` exclusion, and `503 password_safety_unavailable` in `../zunera-backend/tests/Feature/Auth/RegistrationTest.php`
- [ ] T012 [P] [US1] Write failing unit tests for email normalization, password length/spaces/confirmation, local common-password rejection, compromised matches, verifier timeout, and fail-closed outage behavior in `../zunera-backend/tests/Unit/Authentication/EmailNormalizerTest.php` and `../zunera-backend/tests/Unit/Authentication/PasswordRulesTest.php`

### Implementation

- [ ] T013 [US1] Implement normalized registration validation and immutable DTO mapping in `../zunera-backend/app/Http/Requests/Auth/RegisterRequest.php` and `../zunera-backend/app/Data/Authentication/RegisterData.php`
- [ ] T014 [US1] Implement transactional account creation, unique-conflict handling, login, session regeneration, and absolute-expiry metadata in `../zunera-backend/app/Services/Authentication/AuthenticationService.php`
- [ ] T015 [US1] Expose `POST /api/v1/auth/register` with the allow-listed resource and password-safety outage response in `../zunera-backend/app/Http/Controllers/Api/V1/AuthController.php` and `../zunera-backend/routes/api.php`
- [ ] T016 [US1] Run registration/unit tests and verify all registration schemas against `specs/001-user-auth/contracts/auth-api.yaml`

---

## Phase 4: Backend User Story 2 - Sign In and Sign Out (Priority: P1)

**Goal**: Provide sign-in, protected sessions, continuation, expiry, throttling, and current-session logout.
**Independent Test**: Verify valid/invalid credentials, progressive delays, recovery availability, protected denial, warning metadata, both expiry limits, continuation, and logout.

### Tests

- [ ] T017 [P] [US2] Write failing login, session-read, continuation, logout, generic-credential, regeneration, and current-session invalidation tests in `../zunera-backend/tests/Feature/Auth/LoginSessionLogoutTest.php`
- [ ] T018 [P] [US2] Write failing 15-minute idle, one-minute warning, explicit continuation, polling exclusion, and eight-hour absolute-expiry tests in `../zunera-backend/tests/Feature/Auth/SessionLifetimeTest.php`
- [ ] T019 [P] [US2] Write failing five-attempt window, 1/2/4/8/15-minute delay, `Retry-After`, unknown-account equivalence, recovery availability, and success-clear tests in `../zunera-backend/tests/Unit/Authentication/ProgressiveLoginLimiterTest.php`

### Implementation

- [ ] T020 [US2] Implement normalized login validation and DTO mapping in `../zunera-backend/app/Http/Requests/Auth/LoginRequest.php` and `../zunera-backend/app/Data/Authentication/LoginData.php`
- [ ] T021 [US2] Implement HMAC-keyed account/IP throttling and retry metadata in `../zunera-backend/app/Services/Authentication/ProgressiveLoginLimiter.php` and `../zunera-backend/app/Exceptions/LoginThrottledException.php`
- [ ] T022 [US2] Implement login, generic failure, session read/continue, current-session logout, idle tracking, and absolute expiry in `../zunera-backend/app/Services/Authentication/AuthenticationService.php` and `../zunera-backend/app/Http/Middleware/EnforceSessionLifetime.php`
- [ ] T023 [US2] Expose login, session read/continue, and logout routes with `auth:sanctum` protection in `../zunera-backend/app/Http/Controllers/Api/V1/AuthController.php` and `../zunera-backend/routes/api.php`
- [ ] T024 [US2] Run login/session/limiter tests and verify `401`, `429`, `Retry-After`, and expiry schemas against `specs/001-user-auth/contracts/auth-api.yaml`

---

## Phase 5: Backend User Story 3 - Recover Account Access (Priority: P2)

**Goal**: Provide neutral recovery, newest-only one-time reset, all-session invalidation, and safe notifications.
**Independent Test**: Verify known/unknown equivalence, three-email limit, valid reset, expired versus general-invalid codes, reused/superseded denial, old-password/session invalidation, and notification content.

### Tests

- [ ] T025 [P] [US3] Write failing neutral-response, equivalent-path, three-emails-per-hour, queued-delivery, unknown-email, and retry tests in `../zunera-backend/tests/Feature/Auth/PasswordRecoveryRequestTest.php`
- [ ] T026 [P] [US3] Write failing valid, expired, used, superseded, malformed, validation-preserved, concurrent reset, old-password denial, remember-token rotation, all-session deletion, and stable recovery-code tests in `../zunera-backend/tests/Feature/Auth/PasswordResetTest.php`
- [ ] T027 [P] [US3] Write failing encrypted-queue, after-commit, superseded-send suppression, opaque message-correlation, delivery metric, and safe security-notice tests in `../zunera-backend/tests/Unit/Authentication/AuthenticationNotificationTest.php`

### Implementation

- [ ] T028 [US3] Implement normalized recovery/reset Requests and DTOs in `../zunera-backend/app/Http/Requests/Auth/RequestPasswordRecoveryRequest.php`, `../zunera-backend/app/Http/Requests/Auth/ResetPasswordRequest.php`, `../zunera-backend/app/Data/Authentication/RecoveryRequestData.php`, and `../zunera-backend/app/Data/Authentication/ResetPasswordData.php`
- [ ] T029 [US3] Implement neutral queued recovery and the HMAC-keyed three-send rolling-hour limit in `../zunera-backend/app/Services/Authentication/PasswordRecoveryService.php` and `../zunera-backend/app/Services/Authentication/RecoveryEmailLimiter.php`
- [ ] T030 [US3] Implement transactional token validation, expired/general-invalid outcomes, password update, token consumption, remember-token rotation, and session deletion in `../zunera-backend/app/Services/Authentication/PasswordResetService.php`
- [ ] T031 [US3] Implement encrypted queued reset instructions, opaque message-correlation headers, current-token recheck, and password-change notices after commit in `../zunera-backend/app/Notifications/Auth/ResetPasswordNotification.php` and `../zunera-backend/app/Notifications/Auth/PasswordChangedNotification.php`
- [ ] T032 [US3] Expose neutral recovery and reset endpoints with stable recovery/password-safety errors in `../zunera-backend/app/Http/Controllers/Api/V1/AuthController.php` and `../zunera-backend/routes/api.php`
- [ ] T033 [US3] Add explicit recovery error-schema contract assertions in `../zunera-backend/tests/Feature/Auth/PasswordResetContractTest.php`
- [ ] T034 [US3] Run recovery/reset/notification tests and verify all schemas against `specs/001-user-auth/contracts/auth-api.yaml`

---

## Phase 6: Backend Quality and Release Gate

**Purpose**: Finish cross-cutting backend work and prove the contract before frontend starts.

- [ ] T035 Add secret-free auth events and queue/delivery/throttle/expiry metrics in `../zunera-backend/app/Services/Authentication/AuthenticationAudit.php` and `../zunera-backend/config/logging.php`
- [ ] T036 Add recovery-token, session, rate-key, delivery-event, failed-job, and security-log retention schedules in `../zunera-backend/routes/console.php` and `../zunera-backend/.env.example`
- [ ] T037 [P] Create the fixed release timing profile with 10 warm-ups, concurrency five, and the SC-009 10/20/10/20/10/10/10/10 operation mix in `../zunera-backend/scripts/auth-performance-smoke.sh`, `../zunera-backend/tests/Performance/AuthenticationLatencyTest.php`, and `../zunera-backend/tests/Performance/Fixtures/auth-release-profile.json`
- [ ] T038 [P] Write failing canonical delivery-event tests for HMAC signatures, five-minute replay rejection, idempotency, allowed statuses, recipient-data rejection, message correlation, and the 100-message acceptance calculation in `../zunera-backend/tests/Feature/Auth/AuthMailDeliveryEventTest.php` and `../zunera-backend/tests/Feature/Auth/AuthMailDeliveryVerificationTest.php`
- [ ] T039 Implement minimized idempotent delivery-event persistence and the provider-neutral event-source contract in `../zunera-backend/database/migrations/2026_08_30_000002_create_auth_mail_delivery_events.php`, `../zunera-backend/app/Data/Authentication/AuthenticationMailDeliveryEventData.php`, `../zunera-backend/app/Contracts/AuthenticationMailDeliveryEventSource.php`, and `../zunera-backend/app/Services/Authentication/AuthenticationMailDeliveryEvents.php`
- [ ] T040 Implement the signed canonical delivery-event Request/controller/route and 100-message verification command in `../zunera-backend/app/Http/Requests/Auth/StoreAuthenticationMailDeliveryEventRequest.php`, `../zunera-backend/app/Http/Controllers/Api/V1/AuthMailDeliveryEventController.php`, `../zunera-backend/app/Console/Commands/VerifyAuthMailDelivery.php`, and `../zunera-backend/routes/api.php`
- [ ] T041 [P] Document provider delivery events, queue-age alerts, the five-minute threshold, incident steps, and LGPD-safe metrics in `../zunera-backend/docs/auth-mail-operations.md`
- [ ] T042 Validate and reconcile every backend operation, stable error code, status, and schema in `specs/001-user-auth/contracts/auth-api.yaml`
- [ ] T043 Run the full backend PHPUnit suite and Pint and record results in `specs/001-user-auth/quickstart.md`
- [ ] T044 Run the 100-action production-like latency check and record p95 evidence in `specs/001-user-auth/quickstart.md`
- [ ] T045 Run the 100-message production-like provider-delivery check and record delivery-percent evidence in `specs/001-user-auth/quickstart.md`
- [ ] T046 Record a passing backend gate covering contract, authorization, validation, services, security, tests, performance, and mail delivery in `specs/001-user-auth/quickstart.md`

**Checkpoint**: T046 is mandatory before T047 or any other frontend change.

---

## Phase 7: Frontend Foundation (After Backend Gate)

**Purpose**: Build shared design, transport, state, routing, and privacy foundations from the verified backend contract.

- [ ] T047 Add backend base URL and LGPD privacy/privacy-rights URL variables to `../zunera-frontend/.env.example`
- [ ] T048 [P] Write failing CSRF, `419` retry, validation, authentication, throttle, stable recovery-code, password-safety outage, and secret-redaction transport tests in `../zunera-frontend/src/services/__tests__/httpClient.spec.js`
- [ ] T049 Bootstrap Tailwind, Element Plus, semantic Zunera tokens, and the router outlet in `../zunera-frontend/vite.config.js`, `../zunera-frontend/src/main.js`, `../zunera-frontend/src/assets/main.css`, `../zunera-frontend/src/assets/theme.css`, and `../zunera-frontend/src/App.vue`
- [ ] T050 Implement credentialed Axios, one-time CSRF refresh/retry, verified error-code normalization, and secret-safe transport in `../zunera-frontend/src/services/httpClient.js` and `../zunera-frontend/src/services/authService.js`
- [ ] T051 [P] Create the centered `AuthLayout` with mandatory privacy links before form submission in `../zunera-frontend/src/layouts/AuthLayout.vue` and `../zunera-frontend/src/components/auth/PrivacyNoticeLinks.vue`
- [ ] T052 [P] Write layout tests for visible privacy/privacy-rights links, keyboard access, and signed-out content placement in `../zunera-frontend/src/layouts/__tests__/AuthLayout.spec.js`
- [ ] T053 Implement setup-style shared session state and bootstrap without persisting credentials or recovery secrets in `../zunera-frontend/src/stores/auth/sessionStore.js`

---

## Phase 8: Frontend User Story 1 - Create an Account (Priority: P1) 🎯 MVP UI

**Goal**: Register through the verified API and enter protected content.
**Independent Test**: Verify success, validation, duplicate/repeated submit, password-safety outage, protected access, top labels, and privacy links.

### Tests

- [ ] T054 [P] [US1] Write registration response, validation, `503 password_safety_unavailable`, and `name`-absence tests in `../zunera-frontend/src/services/__tests__/authService.registration.spec.js`
- [ ] T055 [P] [US1] Write registration/bootstrap/session-state tests in `../zunera-frontend/src/stores/auth/__tests__/sessionStore.registration.spec.js`
- [ ] T056 [P] [US1] Write top-label, password-guidance, validation, loading-lock, outage, duplicate-feedback, and privacy-link tests in `../zunera-frontend/src/components/auth/__tests__/RegisterForm.spec.js`

### Implementation

- [ ] T057 [US1] Add registration transport and error handling in `../zunera-frontend/src/services/authService.js`
- [ ] T058 [US1] Add registration and authenticated-session actions in `../zunera-frontend/src/stores/auth/sessionStore.js`
- [ ] T059 [US1] Create registration UI using `<ElForm label-position="top">`, `<ElFormItem>`, durable alerts, and one loading state in `../zunera-frontend/src/components/auth/RegisterForm.vue` and `../zunera-frontend/src/components/auth/PasswordRequirements.vue`
- [ ] T060 [US1] Create registration and protected landing views, wrapping the signed-out form in `AuthLayout`, in `../zunera-frontend/src/views/auth/RegisterView.vue` and `../zunera-frontend/src/views/ProtectedHomeView.vue`
- [ ] T061 [US1] Add registration, protected landing, guest-only, and intended-destination routes in `../zunera-frontend/src/router/index.js` and `../zunera-frontend/src/router/authGuard.js`
- [ ] T062 [US1] Add success, duplicate/repeated submit, outage, validation, protected access, 320px, keyboard, and privacy-link Playwright coverage in `../zunera-frontend/e2e/authentication-registration.spec.js`
- [ ] T063 [US1] Run registration Vitest, Playwright, and build checks and record results in `specs/001-user-auth/quickstart.md`

---

## Phase 9: Frontend User Story 2 - Sign In and Sign Out (Priority: P1)

**Goal**: Sign in, manage authoritative session expiry, access protected content, and sign out.
**Independent Test**: Verify credentials, throttle, protected denial, warning/continue, both expiries, logout, top labels, and privacy links.

### Tests

- [ ] T064 [P] [US2] Write login, logout, session, continuation, invalid-credential, throttle, `419`, and `401` transport tests in `../zunera-frontend/src/services/__tests__/authService.session.spec.js`
- [ ] T065 [P] [US2] Write bootstrap, login, logout, invalidation, and authoritative-expiry store tests in `../zunera-frontend/src/stores/auth/__tests__/sessionStore.session.spec.js`
- [ ] T066 [P] [US2] Write top-label, retained-email, cleared-password, alert, loading-lock, and privacy-link tests in `../zunera-frontend/src/components/auth/__tests__/SignInForm.spec.js`
- [ ] T067 [P] [US2] Write warning, focus-trap, keyboard, continue, sign-out, and expiration tests in `../zunera-frontend/src/components/auth/__tests__/SessionExpiryDialog.spec.js`

### Implementation

- [ ] T068 [US2] Add login, logout, session, continuation, throttle, and invalidation methods/actions in `../zunera-frontend/src/services/authService.js` and `../zunera-frontend/src/stores/auth/sessionStore.js`
- [ ] T069 [US2] Create sign-in UI with `<ElForm label-position="top">`, durable feedback, retained email, cleared password, and `AuthLayout` privacy links in `../zunera-frontend/src/components/auth/SignInForm.vue` and `../zunera-frontend/src/views/auth/SignInView.vue`
- [ ] T070 [US2] Implement authoritative timers and accessible warning behavior in `../zunera-frontend/src/composables/useSessionExpiry.js` and `../zunera-frontend/src/components/auth/SessionExpiryDialog.vue`
- [ ] T071 [US2] Enforce protected/guest routes, safe redirects, bootstrap-before-render, and global expiry handling in `../zunera-frontend/src/router/authGuard.js`, `../zunera-frontend/src/router/index.js`, and `../zunera-frontend/src/App.vue`
- [ ] T072 [US2] Add sign-in, invalid credential, protected route, logout, and expiry Playwright journeys in `../zunera-frontend/e2e/authentication-session.spec.js`
- [ ] T073 [P] [US2] Add throttle, recovery availability, keyboard, alert, 200%-zoom, 320px, and privacy-link coverage in `../zunera-frontend/e2e/authentication-security-accessibility.spec.js`
- [ ] T074 [US2] Run session Vitest, Playwright, and build checks and record results in `specs/001-user-auth/quickstart.md`

---

## Phase 10: Frontend User Story 3 - Recover Account Access (Priority: P2)

**Goal**: Request recovery and reset through stable expired/general-invalid outcomes.
**Independent Test**: Verify neutral known/unknown confirmation, newest valid reset, expired/general-invalid feedback, session invalidation, top labels, and privacy links.

### Tests

- [ ] T075 [P] [US3] Write neutral recovery, reset success, stable expired/general-invalid mapping, password-safety outage, and token-redaction tests in `../zunera-frontend/src/services/__tests__/authService.recovery.spec.js`
- [ ] T076 [P] [US3] Write top-label, neutral confirmation, retry, loading-lock, password guidance, recovery states, local-token, and privacy-link tests in `../zunera-frontend/src/components/auth/__tests__/RecoveryAndResetForms.spec.js`

### Implementation

- [ ] T077 [US3] Add recovery/reset transports with stable error mapping and no token persistence in `../zunera-frontend/src/services/authService.js`
- [ ] T078 [US3] Create recovery request/confirmation UI with `<ElForm label-position="top">`, durable neutral feedback, troubleshooting, retry, and `AuthLayout` in `../zunera-frontend/src/components/auth/RecoveryRequestForm.vue`, `../zunera-frontend/src/components/auth/RecoveryConfirmation.vue`, and `../zunera-frontend/src/views/auth/ForgotPasswordView.vue`
- [ ] T079 [US3] Create reset UI with `<ElForm label-position="top">`, password guidance, stable expired/general-invalid states, sign-in path, and `AuthLayout` in `../zunera-frontend/src/components/auth/ResetPasswordForm.vue`, `../zunera-frontend/src/components/auth/RecoveryLinkState.vue`, and `../zunera-frontend/src/views/auth/ResetPasswordView.vue`
- [ ] T080 [US3] Add recovery/reset routes while keeping email/token route-local and out of Pinia/storage in `../zunera-frontend/src/router/index.js`
- [ ] T081 [US3] Add known/unknown recovery, newest-token, expired/general-invalid, reset, old-password, session-invalidation, and privacy-link Playwright journeys in `../zunera-frontend/e2e/authentication-recovery.spec.js`
- [ ] T082 [US3] Run recovery/reset Vitest, Playwright, and build checks and record results in `specs/001-user-auth/quickstart.md`

---

## Phase 11: Frontend Polish and Full Verification

- [ ] T083 [P] Verify every signed-out view renders current privacy/privacy-rights links before form submission in `../zunera-frontend/src/views/auth/__tests__/SignedOutPrivacyLinks.spec.js`
- [ ] T084 [P] Complete keyboard, focus, alert, contrast, light/dark/system, 200%-zoom, and 320px regression coverage in `../zunera-frontend/e2e/authentication-security-accessibility.spec.js`
- [ ] T085 Run the full frontend Vitest suite and record results in `specs/001-user-auth/quickstart.md`
- [ ] T086 Run the frontend production build and record results in `specs/001-user-auth/quickstart.md`
- [ ] T087 Run the isolated Chromium/Firefox/WebKit Playwright suite and record results in `specs/001-user-auth/quickstart.md`
- [ ] T088 Execute all manual authentication smoke scenarios and record outcomes in `specs/001-user-auth/quickstart.md`

---

## Dependencies and Execution Order

1. T001–T010 establish the backend foundation.
2. Backend story slices T011–T016, T017–T024, and T025–T034 may run in parallel after T010, with shared-file edits sequenced.
3. T035–T046 follow all backend stories. T046 is the feature-wide backend gate.
4. T047–T053 start only after T046 and establish shared frontend files.
5. Frontend story slices T054–T063, T064–T074, and T075–T082 follow T053; shared service/store/router edits must be sequenced US1 → US2 → US3.
6. T083–T088 follow all selected frontend stories.

### User Story Dependencies

- Backend US1, US2, and US3 have no cross-story domain dependency after T010.
- Frontend US1, US2, and US3 all depend on the complete backend gate T046 and frontend foundation T053.
- Frontend shared-file changes execute US1 → US2 → US3 even when independent components/tests run in parallel.

## Parallel Examples

```text
Backend after T010: T011/T012 | T017/T018/T019 | T025/T026/T027
Backend release work after T034: T037 | T038 | T041
Frontend foundation after T046: T048 | T051/T052
Frontend tests after T053: T054/T055/T056 | T064/T065/T066/T067 | T075/T076
```

## Implementation Strategy

### MVP

1. Complete the entire backend through T046.
2. Complete frontend foundation T047–T053.
3. Complete frontend US1 T054–T063.
4. Validate registration independently before adding remaining frontend stories.

### Incremental Frontend Delivery

1. Backend is contract-complete once for the feature.
2. Deliver registration UI.
3. Add sign-in/session UI.
4. Add recovery/reset UI.
5. Finish frontend regression verification.

## Notes

- No new Composer or npm packages are required.
- Every Vue form uses `<ElForm label-position="top">` and `<ElFormItem>`.
- Auth JSON exposes user `id` and `email`, never `name`.
- Passwords, recovery tokens, cookies, session IDs, and raw emails never enter logs or persistent frontend storage.
- Commit after each task or coherent task group; avoid unrelated refactors.
