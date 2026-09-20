# Implementation Plan: Credit Cards

**Branch**: `010-credit-cards` | **Date**: 2026-09-20 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/010-credit-cards/spec.md`

**Branch Coordination**: `010-credit-cards` is active in `zunera-specs`,
`../zunera-backend`, and `../zunera-frontend`.

## Summary

Deliver a distinct, owner-scoped credit-card domain. Purchases reserve full
principal, distribute exact-centavo installments into billing cycles, and become
recognized expenses only in their statement-closing month. Statement payments
are dedicated settlement movements: they debit an ordinary financial account but
never become a second categorized expense. Card credit events preserve original
history, adjust statement/budget/reporting projections exactly once, and may
create reusable card credit. Backend contract, authorization, validation,
locking, and tests land before frontend consumption.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13.17+; JavaScript/Vue 3.5.40  
**Primary Dependencies**: Sanctum 4, Eloquent, Axios 1.19, Pinia 4, Element Plus 2.14.5, Tailwind 4.3, Vue Router 5.2, Vue I18n 11.4  
**Storage**: Existing relational financial data plus owner-scoped card records; integer BRL centavos; no stored reporting aggregates  
**Testing**: PHPUnit 12, Laravel Pint, Vitest 4, Playwright 1.61, Redocly  
**Target Platform**: Authenticated Zunera web application  
**Project Type**: Full-stack web feature  
**Performance Goals**: Card list, statement read, and affected financial projections complete within 2 seconds for 10,000 combined owned financial/card movements  
**Constraints**: America/Sao_Paulo business dates; exact centavos; all financial mutations idempotent and atomic; no cash movement at purchase/refund; no card credentials, issuer integration, interest, or recurring-card charge support  
**Scale/Scope**: One owner per card; pagination at existing 50-item page limit; active/archived card history; current zero-value statement; no package additions

## Delivery Scope and Order

Document both scopes in this exact order:

1. **Backend** — Laravel API contracts, authorization, validation, services,
   persistence, and backend tests in `../zunera-backend`.
2. **Frontend** — Vue views, services, Pinia state, Element Plus UI, and frontend
   tests in `../zunera-frontend`.

Frontend implementation MUST start only after the relevant backend API contract,
authorization, validation, and tests are complete.

## Constitution Check

*GATE: Passed before research. Re-checked after Phase 1 design: Passed.*

- [x] Sanctum-protected routes, Form Requests, Resources, owner scopes, and
      no-upload/no-secret constraints protect new boundaries.
- [x] Backend contract, authorization, validation, services, and tests precede
      frontend work; frontend consumes that contract from `../zunera-frontend`.
- [x] Branch `010-credit-cards` matches in specs, backend, and frontend.
- [x] Focused card services/DTOs are justified by exact monetary lifecycle;
      repository abstraction is not.
- [x] Frontend uses JavaScript `<script setup>`, Axios service wrappers, Pinia
      feature state, Element Plus controls, and semantic Tailwind tokens.
- [x] Contract plus backend/unit/frontend/Playwright coverage is planned for
      every lifecycle and cross-feature projection.
- [x] Ownership, idempotency, deterministic locks, no credentials/secrets, and
      explicit scope exclusions are recorded.
- [x] No constitution exception or new package is needed.

## Project Structure

### Documentation (this feature)

```text
specs/010-credit-cards/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── app/
│   ├── Data/CreditCards/
│   ├── Enums/CreditCards/
│   ├── Exceptions/CreditCards/
│   ├── Http/{Controllers/Api/V1,Requests,Resources}/CreditCards/
│   ├── Models/{CreditCard,CreditCardPurchase,CreditCardInstallment,CreditCardStatement,CreditCardStatementPayment,CreditCardCreditEvent,CreditCardCreditApplication,CreditCardMutationRequest}.php
│   └── Services/{CreditCards,Budgets,FinancialDashboard,FinancialHistory}/
├── database/{factories,migrations}/
├── routes/api.php
└── tests/{Feature,Unit}/CreditCards/

../zunera-frontend/
├── src/
│   ├── components/{credit-cards,dashboard}/
│   ├── stores/credit-cards/{creditCardStore.js,__tests__/}
│   ├── services/{creditCardService.js,__tests__/}
│   ├── utils/credit-cards/
│   ├── views/credit-cards/
│   └── {router/index.js,layouts/AppShell.vue,i18n/messages.js}
└── e2e/credit-cards.spec.js
```

**Structure Decision**: `CreditCardService` owns card lifecycle, cycle
allocation, purchases, credits, and idempotency. `StatementPaymentService` owns
statement settlement and delegates only account-balance delta to a focused
reconciler. Projection services own statement/card, Budget, Dashboard, and
Financial History reads. Controllers map validated DTOs to services; Resources
are sole response-shape owners. `CreditCardsListView` and `CreditCardDetailView`
compose focused props/events child components while the Pinia setup store owns
shared fetch/mutation state.

## Delivery Design

### Backend: domain, financial truth, and concurrency

- Credit cards are not `FinancialAccount` records. Add owner-scoped Card,
  Purchase, Installment, Statement, Payment, Credit Event, and Credit
  Application records described in [data-model.md](data-model.md). Card archive
  requires both zero outstanding obligation and zero card credit.
- Store all BRL amounts as integer centavos. Preserve immutable identity
  snapshots on installment/category/card statement history where live archived
  associations can otherwise rewrite past meaning.
- A full purchase amount reserves credit at creation. Installments use
  earliest-installment centavo remainders and statement closing dates as their
  recognition dates. Current active zero statement is a derived read state,
  not a fabricated financial event.
- Business date is America/Sao_Paulo. Closing day is inclusive: a statement
  stays Open throughout that calendar day and finalizes at the start of the next
  America/Sao_Paulo calendar date. Configured dates clamp to month end; due date is first
  configured occurrence after closing. Open/Closed/Partially paid/Paid/Overdue
  state derives from business date and effective settlements/credit applications.
  Existing allocated statements keep original schedule after card billing-day
  update; only later purchases use it.
- Statement payment is a dedicated settlement record and direct financial
  account balance delta, not an ordinary `Transaction` expense. It marks the
  account as financially used, reuses effective/pending/removed lifecycle, and
  is excluded from spending/category/budget/reporting totals. It remains in
  card payment history and normal account balance reconciliation.
- Card credits first adjust their source installments, then automatically apply
  to oldest unpaid statements by due/closing order. Remaining credit is usable
  capacity, separately shown, and may lift available credit above stated limit.
  Payment/credit edits, removal, restore, and correction recalculate all
  applications within one transaction; no statement outstanding amount becomes
  negative.
- Every mutation accepts `Idempotency-Key`, stores owner/key/fingerprint/replay
  response, and uses one transaction. Lock order is financial accounts by id,
  then cards by id, then statements by closing date/id, then affected purchases
  and payments. Recheck availability/outstanding after locks; stale over-limit
  confirmation receives a typed conflict/warning and cannot mutate money. An
  explicit over-limit confirmation is a new mutation with a new key; only a
  network replay of the same initial or confirmed mutation reuses its key.
- Extend derived sources: installments recognized in statement-closing month
  enter Budget/Dashboard/history once when effective; pending open/future
  installments are expected only where existing Budget rules allow it. Payment
  settlement is excluded from those expense totals. Dashboard's ordinary account
  balance stays separate from card obligation; card dashboard data is additive.

### Backend contract and tests

- Add [contracts/credit-cards-api.yaml](contracts/credit-cards-api.yaml) for
  card, purchase, statement, payment, credit-event, dashboard, and history
  shapes. Preserve existing endpoint behavior while extending derived views with
  discriminated card-expense activity and additive card dashboard read data.
- Use Requests, DTOs, Resources, owner-scoped services, typed state exceptions,
  and protected `api/v1` routes. Foreign resources remain indistinguishable
  from absent. Archived cards/accounts/categories remain visible only as retained
  historical associations and cannot be newly selected.
- Unit-test calendar/cycle allocation, centavo distribution, credit and status
  calculation, idempotency fingerprints, deterministic lock ordering, and
  balance/recognition invariants. Feature-test every route, lifecycle,
  authorization, conflict, and source-projection boundary.
- Test 10,000 combined card/financial source rows before adding indexes; inspect
  query plan first. Test close/due day 31, February/leap years, year rollover,
  closing-day purchase, 1/360 installments, payment/refund reallocation, card
  credit, archived associations, and concurrent payment/purchase requests.

### Frontend: views, state, and accessibility

- Add authenticated lazy routes for active cards, archived cards, and card detail
  under `/app`; place the distinct Credit Cards primary navigation item beside
  financial-account management without reusing account components. Route metadata
  drives title and active navigation.
- `creditCardService` centralizes CSRF, idempotency keys, data unwrap, and
  field-error mapping. It creates a fresh key for an explicit over-limit
  confirmation and reuses a key only for network replay of the identical
  mutation. Feature Pinia setup store uses shallow refs for server
  snapshots/loading/errors and explicit actions. It never calculates money,
  status, cycle, or available credit locally; after mutation it refreshes only
  impacted card/statement plus affected dashboard/budget/history slices.
- Use `CreditCardsListView` for scan-friendly summary/list/lifecycle controls;
  `CreditCardDetailView` composes detail summary, statement list/detail drawer,
  purchase list, and payment/credit event history. Focused dialogs provide card,
  purchase, payment, and credit-event forms; dedicated confirmation handles
  archive/cancellation/removal.
- All child components receive immutable props and emit intent. Use computed for
  derived display/action availability, watchers only for effects, pure BRL/date/
  installment/status formatters in utilities, and no API/domain logic in views.
- Use `<ElForm label-position="top">`, `CurrencyAmountInput`, `ElTable` with
  responsive column priority/detail access, `ElDialog`/drawer focus return,
  `ElAlert`/tags/textual statuses, `ElEmpty`, and durable post-save content.
  Clear Element Plus validation on dialog close and prevent repeated submits.
- Respect semantic Tailwind tokens, Light/Dark/System, 16/24/32px shell gutters,
  44px targets, one h1, 320px, 200% zoom, reduced motion, keyboard flow, and
  non-color financial/status cues. Add pt-BR/en translations.

### Test strategy and delivery sequence

1. **Backend first**: migrations/factories/enums/models; cycle/money/credit and
   settlement services; Requests/DTOs/Resources/controllers/routes; contract;
   unit/feature/performance tests. Verify account effects and all projections
   before any frontend contract consumer.
2. **Frontend second**: routes/navigation/i18n; Axios service + Pinia store;
   list/detail/dialog components and formatters; service/store/component/view
   tests; Playwright critical user journeys against contract-backed mocks.
3. **Cross-feature regression**: Budget statement-month recognition,
   Dashboard expense/result and separate obligation display, Financial History
   card-expense entries, unchanged ordinary account overview, and no payment
   double count.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| N/A | N/A | N/A |
