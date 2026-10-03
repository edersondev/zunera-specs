# Implementation Plan: Account Activation

**Branch**: `019-account-activation` | **Spec**: [spec.md](spec.md)

## Constitution Check

- [x] Backend contract, validation, authorization, service, migration, and tests
  precede frontend work.
- [x] Specs, backend, and frontend use matching feature branches.
- [x] Existing user access is preserved; activation tokens are hashed in the
  activation table and never logged.
- [x] Frontend consumes the backend contract, uses Vue Composition API, and
  follows the shared design documents.
- [x] Contract, feature, notification, UI, and browser checks are included.

The checks held before backend design and were reviewed again after frontend
design. The canonical HTTP contract is
[auth-api.yaml](../001-user-auth/contracts/auth-api.yaml).

## Backend first

1. Backfill `email_verified_at` for existing users and create a one-token-per-user activation table containing only a token hash and issue time. Apply migration before enabling new registration behavior.
2. Change registration to return `201 {message, activation_required: true}` without starting a session. Queue a localized activation notification after commit. Update the canonical auth API contract.
3. Add `POST /api/v1/auth/activation/confirm` with `{email, token}` and `POST /api/v1/auth/activation/resend` with `{email}`. Confirmation consumes only the latest unexpired token; resend is neutral and limited to three sends per address and 20 per IP per hour.
4. Reject correct inactive credentials with `403` and `code: account_inactive`; retain generic `401` for wrong credentials. Apply active-account middleware to the protected API group.
5. Add an `auth-mail` queue worker to the development stack, PHPUnit feature and notification tests, then run Pint and focused auth tests.

## Frontend after backend contract

1. Update auth service and Pinia registration flow to expect a confirmation instead of a session.
2. Show a check-email result after registration. Add an activation route and result view that clears the token from the URL before posting it, and offers resend and sign-in actions.
3. Add inactive-account guidance to sign-in. Localize new copy in PT-BR and English and follow the design foundation, app shell, navigation, and component catalog.
4. Run focused Vitest, a Playwright activation journey, build, and lint.

## Rollout

Deploy and run the migration before serving the new backend flow. Keep the `auth-mail` worker running and inspect queue failures after release. The development stack migration was applied and its worker started during implementation.

## Documentation Validation

- Parsed the OpenAPI 3.1 YAML with duplicate-key detection and resolved every
  internal `$ref`.
- Matched registration, confirmation, resend, inactive-login, and activation-link
  status and response shapes to the backend controller, request rules, exceptions,
  and feature tests.
- Checked the relative Markdown links and `git diff --check` before commit.
