# Quickstart: Category Management Feature

## Backend first

1. In `../zunera-backend`, confirm branch `003-category-management`.
2. Do not add packages. Implement the protected contract in
   `contracts/categories-api.yaml` before any frontend work.
3. Add persisted, idempotently seeded system defaults and personal-category
   persistence; use a focused category service, Form Requests, DTOs, API
   Resources, and typed conflict handling consistent with financial accounts.
4. Enforce active normalized personal-name uniqueness, system-default conflict
   checks, read-only system behavior, personal ownership, archive/restore, and
   classification locking after use.
5. Add feature and unit coverage. Then run migrations, inspect routes, run
   focused tests, full backend tests, Pint, and contract lint.

## Frontend after backend gate

1. In `../zunera-frontend`, confirm branch `003-category-management`.
2. Consume only the confirmed `contracts/categories-api.yaml` contract through
   a feature Axios service and setup-style Pinia store.
3. Add authenticated active and archived category routes, category navigation,
   and focused list, form, and lifecycle-dialog components. Active results show
   defaults and personal categories; archived results show only personal
   categories.
4. Follow `docs/design/design-foundation.md`, `docs/design/app-shell.md`,
   `docs/design/navigation.md`, and `docs/design/components.md`: semantic
   colors, visible labels, accessible controls, loading/empty/error/success
   states, and responsive layouts.
5. Defaults show a non-color “System default” marker and no mutation actions.
   A used personal category shows classification as locked with explanatory
   text. Archive and restore require confirmation and durable state feedback.
6. Add service, store, view/component, and isolated Playwright coverage before
   running unit tests, build, and category journeys.

## Manual acceptance smoke test

- Sign in with no personal categories; verify all 16 system defaults are
  visible, active, labeled as system defaults, and separated by income/expense.
- Create an expense category named `Pet care`; verify its personal origin,
  classification, optional visual identity, and durable success feedback.
- Try another `PET CARE` with surrounding or repeated spaces, and a personal
  `Food` expense category; verify clear conflict feedback and no duplicate.
- Update unused personal category name, classification, color, and icon; verify
  valid updates. Simulate a used category; verify classification lock feedback
  while allowed visual/name edits remain possible.
- Attempt to edit, archive, or restore a system default; verify clear read-only
  feedback and no mutation.
- Create categories for two users; verify neither user can view or mutate the
  other's personal category.
- Archive a personal category flagged as used; verify it disappears from normal
  selection, remains listed as archived, and its record is preserved. Restore
  it and verify it returns to active selection unless a conflict exists. Actual
  transaction-association retention is verified by the future transaction
  feature that introduces those records.
- Verify no category deletion action is offered.
- Verify Light, Dark, and System themes; 320px viewport; 200% zoom;
  keyboard-only use; dialog focus restoration; and readable non-color labels.

## Verification commands

```sh
php artisan route:list -vv --path=api
php artisan test --compact tests/Feature/Categories tests/Unit/Categories
php artisan test --compact
vendor/bin/pint --dirty --format agent
npx @redocly/cli lint specs/003-category-management/contracts/categories-api.yaml
npm run test:unit -- --run
npm run build
CI=1 npm run test:e2e -- e2e/categories.spec.js
```

## Planning evidence

- [x] Contract identifies authentication, ownership-safe not-found behavior,
  validation feedback, state conflicts, and resource responses.
- [x] Data model distinguishes global system defaults from user-owned personal
  categories and captures forward-compatible transaction-use locking.
- [x] Research records default, lifecycle, uniqueness, and visual-semantic
  decisions.
- [x] Implementation verification completed: Redocly contract lint; backend
  route listing and Pint; Docker-focused category backend tests (14 tests, 74
  assertions); Docker full backend suite (69 tests, 276 assertions); focused
  category frontend unit tests (9 files, 10 tests); frontend production build;
  and Chromium category Playwright journeys (2 passed).
