# Quickstart: Financial Goals

## Read before implementation

1. Verify `015-financial-goals` is active in specs, `../zunera-backend`, and `../zunera-frontend` before editing each repository.
2. Read [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [the API contract](contracts/financial-goals-api.yaml), and the four design files in `docs/design/`.
3. Complete backend API contract, authorization, validation, service behavior, and tests before frontend implementation. No new package is planned.

## Backend path

1. Add goal, append-only activity, and goal mutation request persistence. Keep them separate from financial-account balances, Transactions, Transfers, financial history, Budget, and cards.
2. Add owner-scoped protected routes, Form Requests, DTOs, Resources, domain errors, and thin controllers matching the documented contract. Require `Idempotency-Key` for every mutation.
3. Implement goal service operations and atomic activity writes. For linked allocations and reassociation, lock the goal and relevant account rows, check current capacity, and reject the whole action on conflict. Never invoke a balance reconciler from a goal mutation.
4. Add overview, detail, activity, account shortfall, monthly guidance, and independent Dashboard goals projections. Account-free allocations remain marked unverified. Keep previous Dashboard cash-flow and budget calculations unchanged.
5. Prove behavior with feature/unit tests and contract checks before frontend begins. Include actual overlapping requests against MySQL for shared-account capacity; SQLite in-memory unit tests alone cannot prove row-lock behavior.

Preferred backend checks from `../zunera-backend/` while the development stack runs:

```bash
docker exec zunera-backend-app-1 php artisan test --filter=FinancialGoal
docker exec zunera-backend-app-1 php artisan test --filter=FinancialDashboard
docker exec zunera-backend-app-1 php artisan test --filter=Budget
docker exec zunera-backend-app-1 php artisan test --filter=CreditCard
docker exec zunera-backend-app-1 php artisan test
docker exec zunera-backend-app-1 vendor/bin/pint --format=agent
```

Docker socket access may require sandbox escalation. Check fresh and upgraded schema behavior in the live MySQL stack. Confirm the implemented request and response shapes match `contracts/financial-goals-api.yaml`.

## Frontend path

1. Add a goals service around `apiRequest`, a setup-style Pinia store for shared overview/detail state, and independent Dashboard goals fetching. Reuse retry-safe idempotency-key behavior for mutations and refresh server-derived figures after success.
2. Add protected overview, detail, and archived-history routes plus navigation. Compose small goal cards, form/amount dialogs, progress, account backing, and history components. Use `CurrencyAmountInput`, visible Element Plus form labels, existing theme tokens, and escaped text rendering.
3. Show account balance, designated and unallocated amounts separately; show shortfall and unverified backing in words and values. Keep completed actions locked until reopen; require full withdrawal before archive; permit restore at zero.
4. Add PT-BR/English copy, keyboard/assistive-technology semantics, 320 px/200% zoom, light/dark/system themes, and independent Dashboard error/retry state. Verify overfunded progress announces actual percentage even when visual fill stops at 100%.
5. Add service/store/component tests and isolated Playwright journeys. Verify navigation and dashboard goals without changing cash-flow figures.

Frontend checks from `../zunera-frontend/`:

```bash
npm run test:unit -- --run
npm run lint
npm run build
CI=1 npm run test:e2e -- e2e/financial-goals.spec.js
CI=1 npm run test:e2e -- e2e/financial-dashboard.spec.js
```

## Acceptance walkthrough

1. Start with an account at R$ 10.000,00. Create a R$ 30.000,00 goal with R$ 3.000,00 initial allocation. Verify R$ 3.000,00 goal allocation, R$ 7.000,00 account unallocated amount, R$ 10.000,00 actual account balance, and no new financial-history, budget, or income/expense entry.
2. Add R$ 500,00, withdraw R$ 200,00, retry each request with the same key, and verify exactly one event per action and R$ 3.300,00 current allocation. Reuse a key with different payload and verify 409 without change.
3. Link a second goal to the account and use overlapping requests to allocate its remaining capacity. Only one request may claim capacity beyond the shared limit. Record a real expense afterward that lowers actual balance below total allocations; verify exact shortfall warning, no auto-withdrawal, and no new linked addition until capacity returns.
4. Create an account-free goal with a positive initial allocation; verify the unverified label and separate unverified overview subtotal. Change it to a funded account only when that account has sufficient unallocated capacity. Change association again; verify no transfer and prior event account snapshots intact.
5. Reach and exceed a target; verify true progress above 100%, full allocation, zero remaining, and excess value. With a linked-account shortfall or inactive/unavailable association, reject completion; resolve the shortfall or relink/unlink, then complete manually. Create a later real shortfall or archive the associated account and verify completed status remains with attention visible. Reject withdrawal until reopen, then withdraw all, archive, restore, and verify zero allocation with all history retained. Attempt archive while funded and verify explicit rejection. An unlinked target-funded goal may complete with an unverified label.
6. Set a future date and verify rounded monthly guidance over inclusive remaining calendar months. Verify due-today/overdue and reached-target cases show no suggestion; no recurring transaction is created.
7. Archive a linked account and confirm the goal remains readable, additions stop, and withdrawal/reassociation remain possible. Compare Financial Accounts, Transfers, Transactions, Credit Cards, Budget, financial history, and Dashboard before and after goal-only actions for no duplicate financial recognition.

## Outcome measurement

- For the two-second goal, seed 100 owned goals with 1,000 mixed activity entries and account links. Take at least 20 warm browser timings each for overview and detail on representative desktop and 320 px mobile viewport; record p95 from navigation to usable primary content. Inspect backend query counts to catch per-goal activity/account fetches.
- For cent-level and no-double-count criteria, run fixture-driven cases for every named monetary/lifecycle boundary and compare goal projections with authoritative account, Dashboard, Budget, card, and financial-history outputs. Run overlapping MySQL-backed requests, not only sequential calls.
- For SC-001 and SC-004, recruit at least 10 representative users and give each uncoached create-goal and shortfall-identification tasks. Record each creation time, whether it finished within 3 minutes, whether the user identified the shortfall and distinguished actual/designated/unallocated amounts, and whether the user found a corrective action. At least 90% must pass each criterion. Record anonymized results in `checklists/usability.md`; fix and repeat a failed check before delivery. Do not collect financial credentials.
