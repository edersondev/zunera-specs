# Quickstart: Authentication Feature

## Full name and locale update

- Registration requires a trimmed `name` between 2 and 255 characters.
- Register, login, and session responses include `data.user.id`, `name`, and `email`.
  Only pre-existing accounts can return `name: null`; clients fall back to email.
- Send `Accept-Language: pt-BR` or `Accept-Language: en` for localized API feedback.
  Missing and unsupported values safely use PT-BR.
- Frontend defaults to PT-BR and keeps account-menu language selection in local storage.

## Backend-first setup

1. In `../zunera-backend`, check out branch `001-user-auth` and install the already-declared dependencies.
2. Configure the database, DB session/cache/queue, production transactional email gateway, canonical delivery-event URL, HMAC-SHA256 secret, ±300-second replay window, `FRONTEND_URL`, credentialed CORS origins, and Sanctum stateful domains through environment variables.
3. Run migrations. Existing nullable `users.name` accounts remain compatible; new registrations require a name.
4. Start the queue worker and mail service. Recovery and security notifications are queued after transaction commit.
5. Run focused PHPUnit tests, then the full suite and Pint.
6. Complete all backend stories, the OpenAPI check, the 100-action latency check,
   and the 100-message delivery-event acceptance check before changing frontend files.

## Frontend after backend contract

1. In `../zunera-frontend`, check out branch `001-user-auth`.
2. Set the API base URL and credentialed Axios configuration. Call `/sanctum/csrf-cookie` before mutating auth requests.
3. Implement the route guard and auth store only from `contracts/auth-api.yaml`; keep passwords and reset secrets component-local.
4. Run unit tests, build, then Playwright authentication and accessibility journeys.

## Manual acceptance smoke test

- Register with a 15+ character password; verify immediate protected access.
- Sign out, sign in with an invalid password, and verify generic feedback, retained email, and cleared password.
- Request recovery for both known and unknown addresses; verify identical neutral confirmation.
- Use the newest reset link, then verify old sessions end and the security email is sent.
- Verify the one-minute idle warning, continue action, 15-minute idle cutoff, and 8-hour absolute cutoff.
- Verify five failed sign-ins trigger progressive delays and recovery remains available.

## Verification commands

```sh
php artisan route:list -vv --path=api
php artisan test --compact
vendor/bin/pint --dirty --format agent
scripts/auth-performance-smoke.sh --profile=release --warmup=10 --samples=100 --concurrency=5 --timing=client-monotonic
php artisan auth-mail:verify-delivery --samples=100
npm run test:unit -- --run
npm run build
npm run test:e2e
```

## Implementation evidence

Full-name and bilingual update, recorded 2026-09-11:

- Frontend unit suite passed: 58 tests in 31 files.
- Frontend production build passed.
- Backend Pint and PHP syntax checks passed.
- Focused backend feature tests could not start in this checkout because the local
  PHP runtime has no `pdo_sqlite` driver; all ten database tests stop during
  `RefreshDatabase` setup before assertions. Run them in a PHP image with
  SQLite enabled or against the project test database.

Recorded 2026-08-30:

- Backend routes compile in Docker with eight authentication API routes under
  `/api/v1`.
- Backend migrations ran in Docker with `php artisan migrate --force`; all six
  migrations report `Ran`, including nullable `users.name` and
  `auth_mail_delivery_events`.
- Focused backend authentication suite passed in Docker:
  `php artisan test --compact tests/Unit/Authentication tests/Feature/Auth tests/Performance`
  reported 24 passing tests.
- Full backend suite passed in Docker:
  `php artisan test --compact` reported 25 passing tests.
- Laravel Pint ran in Docker with `vendor/bin/pint --format agent`; it formatted
  three changed PHP files. `--dirty` could not be used inside the container
  because the mounted container path does not expose Git metadata.
- OpenAPI contract lint passed:
  `npx @redocly/cli lint specs/001-user-auth/contracts/auth-api.yaml`.

Pending release evidence before frontend work:

- SC-009 100-action production-like latency check with monotonic client timing.
- 100-message provider delivery-event check proving at least 95 delivered
  canonical events within five minutes.
- Backend gate T046 remains open until both release checks are recorded.
- Frontend implementation proceeded after explicit owner waiver on 2026-08-30;
  T044-T046 remain open and must be completed before release.

Frontend evidence recorded 2026-08-30:

- Frontend uses local machine commands per owner instruction.
- Unit suite passed:
  `npm run test:unit -- --run` reported 13 passing files and 16 passing tests.
- Production build passed with Vite:
  `npm run build`.
- ESLint passed without cache:
  `npx eslint . --fix`. The package script with `--cache` could not write
  `.eslintcache` on the read-only mounted path.
- Oxlint passed:
  `npm run lint:oxlint`.
- Playwright browser binaries were installed with `npx playwright install`.
- Chromium and Firefox E2E passed:
  `CI=1 npm run test:e2e -- --project=chromium --project=firefox`
  reported 10 passing tests.
- Full Playwright remains open because WebKit cannot launch on this host without
  missing system dependency `libavif16`; `npx playwright install-deps` failed.
