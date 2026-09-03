# Quickstart: Financial Accounts Feature

## Backend-first setup

1. In `../zunera-backend`, confirm branch `002-financial-accounts`.
2. Install already-declared dependencies; do not add new packages.
3. Implement the backend contract in `contracts/financial-accounts-api.yaml`
   first: protected account routes, ownership authorization, Form Requests,
   Resource output, service-layer rules, account persistence, lifecycle actions,
   centavo money handling, and tests.
4. Run migrations and verify account routes.
5. Run focused PHPUnit tests for financial accounts, then the full backend suite
   and Pint.
6. Lint the OpenAPI contract before frontend work starts.

## Frontend after backend contract

1. In `../zunera-frontend`, confirm branch `002-financial-accounts`.
2. Implement only from `contracts/financial-accounts-api.yaml`.
3. Add the financial-account service, setup-style Pinia store, protected routes,
   account views, feature components, currency formatting utilities, and tests.
4. Follow `docs/design/design-foundation.md`, `docs/design/app-shell.md`,
   `docs/design/navigation.md`, and `docs/design/components.md`.
5. Run unit tests, build, and Playwright account journeys.

## Manual acceptance smoke test

- Sign in and create a checking account with `R$ 1.250,50`; verify success,
  active list visibility, current balance, and active combined balance.
- Create an account named `Conta principal` with institution `Nubank`; verify
  the account name and financial institution appear as separate values.
- Create accounts for two different users; verify neither user can view or change
  the other's accounts.
- Try creating a second active account with the same name for one user; verify
  name-conflict feedback, including names that differ only by spaces, case, or
  accents.
- Update name, account type, institution text, predefined color, predefined icon,
  and eligible initial balance; verify durable success feedback and detail view.
- Try `R$ 9.999.999.999,99`, `-R$ 9.999.999.999,99`, and one cent beyond each
  limit; verify accepted boundary values and rejected out-of-range values.
- Archive an active account; verify it disappears from normal active choices and
  active combined balance but remains visible in archived/detail views.
- Archive the last active account; verify the active list is empty and active
  combined balance is zero.
- Restore an archived account; verify it returns to active lists unless its name
  conflicts with an active account.
- Attempt to edit initial balance after a simulated account with movements;
  verify locked-field state feedback.
- Verify permanent deletion is not offered.
- Verify Light, Dark, and System themes; 320px viewport; 200% zoom; keyboard-only
  navigation; and screen-reader labels for primary account journeys.

## Verification commands

```sh
php artisan route:list -vv --path=api
php artisan test --compact tests/Feature/FinancialAccounts tests/Unit/FinancialAccounts
php artisan test --compact
vendor/bin/pint --dirty --format agent
npx @redocly/cli lint specs/002-financial-accounts/contracts/financial-accounts-api.yaml
npm run test:unit -- --run
npm run build
CI=1 npm run test:e2e -- --project=chromium --project=firefox --project=webkit e2e/financial-accounts.spec.js
```

## Release evidence recorded

- [x] Backend route list includes all versioned financial-account routes.
- [x] OpenAPI contract lint passes with Redocly.
- [x] Backend focused suite passes (30 tests) and full backend suite passes
  (55 tests) in the project container.
- [x] Laravel Pint passes; it formatted only the intended financial-account
  files.
- [x] Frontend unit suite passes (44 tests) and the production build passes.
- [x] Playwright financial-account journeys pass in Chromium and Firefox.
  WebKit is documented as blocked by the host missing `libavif16`, so the
  narrower passing browser set is recorded as allowed by this quickstart.
- [x] Compact-width, dark-theme, keyboard-focus, and 200% zoom checks pass for
  the financial-account shell; accessible selectors and isolated scenarios are
  used by the Playwright journeys.
