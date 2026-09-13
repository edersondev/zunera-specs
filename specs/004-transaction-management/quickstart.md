# Quickstart: Transaction Management Feature

## Backend first

1. In `../zunera-backend`, confirm branch `004-transaction-management`.
2. Do not add packages. Implement the protected contract in
   `contracts/transactions-api.yaml` before any frontend work.
3. Add transaction persistence with integer centavos, a two-value financial
   status, and a nullable removal timestamp. Use a focused transaction service,
   Form Requests, DTOs, API Resources, and typed state conflicts consistent with
   financial accounts and categories.
4. Enforce ownership, active-versus-archived association rules, type-and-
   category compatibility, amount and date bounds, the future-dated pending
   rule, remove/restore lifecycle, and the paginated filter/search contract.
5. Reconcile account balances by delta inside the same database transaction as
   every create, update, status change, account change, type change, removal,
   and restoration, locking the affected account rows.
6. Require and persist a unique `Idempotency-Key` for every transaction
   mutation. Matching retries must replay the original response; the same key
   with different payload must return `409 idempotency_key_reused`.
7. Set `financial_accounts.has_financial_movements` and
   `categories.has_financial_transactions` on first association, keeping the
   locks promised by those features.
8. Add feature and unit coverage. Then run migrations, inspect routes, run
   focused tests, full backend tests, Pint, and contract lint.

## Frontend after backend gate

1. In `../zunera-frontend`, confirm branch `004-transaction-management`.
2. Consume only the confirmed `contracts/transactions-api.yaml` contract through
   a feature Axios service and setup-style Pinia store.
   Generate one `Idempotency-Key` per logical save/remove/restore action and
   reuse it only when retrying that exact action.
3. Add authenticated transaction routes: history with filters and search, a
   record form, an edit path, a details view, and a removed-transactions view
   with restore.
4. Follow `docs/design/design-foundation.md`, `docs/design/app-shell.md`,
   `docs/design/navigation.md`, and `docs/design/components.md`: financial
   positive/negative semantic tokens for income and expense, visible labels and
   signs so meaning never depends on color, accessible controls, and
   loading/empty/error/success states.
5. Show newest first with progressive loading in batches of at most 50 and the
   count of matching transactions. Report active filters and allow clearing
   them. Archived accounts and categories appear as read-only labels.
6. Add service, store, view/component, and isolated Playwright coverage before
   running unit tests, build, and transaction journeys.

## Manual acceptance smoke test

- Sign in with at least one active account and income/expense categories.
  Record an income transaction and an expense transaction; verify each appears
  in history with description, amount, type, date, account, category, and
  status, and that both account balances changed by exactly the right amount.
- Record a transaction dated in the future; verify it is stored as pending, does
  not change any balance, and can be marked effective later, at which point the
  balance changes exactly once.
- Record a transaction with a note; verify the note persists and appears in the
  detail view, and that searching a word from the note finds it.
- Edit amount, type, category, account, date, and status in turn; verify the
  previously affected account and the newly affected account both end at the
  correct balance, with no stale or duplicated effect.
- Edit an effective transaction's date into the future; verify it stays
  effective, keeps affecting the balance, and shows a notice offering to set it
  pending.
- Retry a completed create, update, remove, and restore request with the same
  `Idempotency-Key`; verify the original response is replayed and balances move
  only once. Reuse a key with a changed payload and verify the typed 409.
- Attempt invalid input: zero, negative, over-precision, or over-limit amounts;
  a date outside 1900-01-01..2100-12-31; a blank or overlong description; an
  income transaction with an expense category and vice versa. Verify no
  transaction is stored and feedback identifies the field to correct.
- Attempt to use another user's account, category, or transaction; verify
  access is denied without revealing that data.
- Remove an effective transaction; verify the balance is reversed, the
  transaction leaves the normal history, and it appears in the removed view.
  Restore it as effective; verify the balance is applied exactly once and
  removal/restoration feedback is shown.
- Archive an account and a category that have transactions; verify those
  transactions remain visible with read-only archived labels, remain editable
  while keeping that association, and that archived resources cannot be chosen
  for any other transaction. Include a pending transaction in each case and
  verify archiving is not blocked, the pending state and zero balance effect are
  unchanged, and the transaction stays visible and editable.
- Filter by date range, type, account, category, and status, combine filters,
  and search text; verify only matching transactions are listed, the matching
  count and active criteria are visible, and no-match shows a clear empty state.
  Attempt account/category filters using another user's identifiers; verify a
  privacy-safe 404 with no data exposed.
- Verify a 5,000-transaction history loads newest first, that the first batch
  appears quickly, and that the classic "identical duplicate" pair is allowed
  while a repeated submit of one action does not double-apply a balance effect.
- Verify Light, Dark, and System themes; narrow supported viewports; 200% zoom;
  keyboard-only use; accessible dialog and drawer behavior; and that income and
  expense stay distinguishable without color.

## Verification commands

```sh
# Backend
php artisan migrate
php artisan route:list -vv --path=api
php artisan test --compact tests/Feature/Transactions tests/Unit/Transactions
php artisan test --compact
vendor/bin/pint --dirty --format agent

# Contract
npx @redocly/cli lint specs/004-transaction-management/contracts/transactions-api.yaml

# Frontend
npm run test:unit -- --run
npm run build
CI=1 npm run test:e2e -- e2e/transactions.spec.js
```

## Planning evidence

- [x] Contract identifies authentication, ownership-safe not-found behavior,
  validation feedback, state conflicts, paginated history with a matching
  count, and resource responses with embedded account and category summaries.
- [x] Data model documents the two financial states, the removal lifecycle,
  amount and date bounds, association rules, and the exact balance
  reconciliation formula.
- [x] Research records the centavos, materialized-balance, status, removal,
  ownership, archived-association, pagination, search, and history-flag
  decisions with rejected alternatives.
- [x] Implementation verification complete: branch confirmation, migrations,
  route listing, focused and full backend tests, Pint, Redocly contract lint,
  frontend unit tests, production build, isolated Playwright transaction
  journeys, manual acceptance smoke test, and visual/accessibility review.

## Implementation notes

- Frontend mutations resolve to `{ transaction, meta }` so the store can surface
  the typed `meta.notice` from update and create responses without re-parsing the
  transport envelope.
- After every transaction mutation the store refreshes the list and the
  financial-account store, then reports the per-account balance delta
  (`lastBalanceImpact`) as visible success feedback. Archived accounts are not
  part of the active account collection, so their feedback relies on the
  refreshed list rather than that delta.
- Clearing the transaction filters resets every criterion, not only `view` and
  `per_page`, because the store merges the reset payload over the current
  filters.
- Archived account and category associations remain selectable while they are
  the transaction's current association and are labelled `(arquivada)` in the
  form, history, and detail views.

## Implementation verification evidence

- Backend focused suite: 32 tests, 314 assertions passing
  (`php artisan test --compact tests/Feature/Transactions tests/Unit/Transactions`).
- Backend full suite: 110 passing, 0 failing. Three stale auth assertions that
  relied on an implicit PT-BR locale now request `Accept-Language: pt-BR`
  explicitly, and `SetRequestLocale` was corrected so a missing or unsupported
  header really falls back to PT-BR as FR-026 and the auth quickstart require.
- Backend style: `vendor/bin/pint --test --format agent` passes.
- Frontend unit suite: 42 files, 98 tests passing; `npm run build` succeeds.
- Frontend e2e: 5 transaction journeys pass on Chromium, Firefox, and WebKit
  (15/15) with `CI=1 npm run test:e2e -- e2e/transactions.spec.js`. WebKit needed
  the host Playwright dependencies (`libavif16`, `libwoff1`, `xvfb`).
- Manual acceptance smoke test and Light, Dark, System, narrow-viewport,
  200%-zoom, keyboard, and focus-restoration verification completed.
