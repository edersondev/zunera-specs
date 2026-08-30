# Quickstart: Authentication Feature

## Backend-first setup

1. In `../zunera-backend`, check out branch `001-user-auth` and install the already-declared dependencies.
2. Configure the database, DB session/cache/queue, production transactional email provider, `FRONTEND_URL`, credentialed CORS origins, and Sanctum stateful domains through environment variables.
3. Run migrations. The feature migration makes `users.name` nullable; no seed account is required.
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
scripts/auth-performance-smoke.sh --samples=100
php artisan auth-mail:verify-delivery --samples=100
npm run test:unit -- --run
npm run build
npm run test:e2e
```
