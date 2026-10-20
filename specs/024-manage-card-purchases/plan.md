# Implementation Plan: Manage Credit Card Purchases

**Branch**: `024-manage-card-purchases` | **Date**: 2026-10-20 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/024-manage-card-purchases/spec.md`

**Branch Coordination**: `024-manage-card-purchases` is active in specs, backend, and frontend. The existing unrelated frontend `package-lock.json` change is left untouched.

## Summary

Extend manual card purchase creation with a server-validated statement choice when its date-derived cycle has closed. Add previewed moves for a full installment schedule or one installment, including closed and paid statements. Add confirmed permanent deletion, blocked when any refund or correction credit is linked. Preserve payment and account cash history while a card-wide reconciliation recalculates statement obligations, card credit, spending, and notifications. Backend contract and tests precede Vue work.

## Technical Context

**Language/Version**: PHP 8.3 / Laravel 13.17; JavaScript / Vue 3.5 with `<script setup>`
**Primary Dependencies**: Existing Sanctum/session auth, Eloquent, Element Plus 2.14, Pinia 4, Axios 1.19, Tailwind CSS 4, vue-i18n 11
**Storage**: Existing relational card tables; SQLite test environment; new move-history and payment-surplus allocation tables plus small lifecycle columns
**Testing**: PHPUnit feature/unit in backend with deterministic 100-case financial/date/retry matrices; Vitest service/component/store with a 100-case form matrix; Playwright isolated journeys; Pint and frontend lint; scripted 20-user usability check
**Target Platform**: Authenticated responsive web app, pt-BR/en, São Paulo business calendar, Light/Dark/System themes
**Project Type**: Full-stack financial feature across sibling Laravel API and Vue client
**Performance Goals**: Windowed statement-option queries and preview; p95 option/preview responses under 2 seconds on representative cards with 1,000 purchases; no unbounded cycle enumeration
**Constraints**: No new packages, no new cash movement from reassignment/deletion, exact centavos, permanent purchase removal, retained payment history, owner isolation, idempotent mutations
**Scale/Scope**: Up to 360 installments per purchase; any eligible empty future cycle can be selected through direct month lookup; three purchase journeys and affected card projections

## Delivery Scope and Order

1. **Backend** (`../zunera-backend`) — Define and test statement-option, create, move-preview, move, and delete contracts. Add Form Requests, DTOs, owner checks, transactional services, persistence for move history and payment surplus, card-wide reconciliation, Resource responses, recurrence tombstone, and financial projection coverage. Finish backend API contract, authorization, validation, and focused tests before frontend work.
2. **Frontend** (`../zunera-frontend`) — Consume the settled contract through `creditCardService` and the existing credit-card Pinia store. Add an explicit Statement select for closed-cycle manual purchases, focused move/delete dialogs, actions on purchase and statement views, translations, accessibility, and service/component/Playwright coverage.

## Constitution Check

*Gate evaluated before research and again after design: PASS. No exception is needed.*

- [x] Existing authenticated API middleware remains the boundary; new Form Requests validate owner, date, mode, target, and idempotency inputs; API Resources own response shapes. No uploads.
- [x] Backend contract, authorization, validation, reconciliation, and tests are complete before frontend implementation starts.
- [x] All three repositories use `024-manage-card-purchases`; no application edit is planned on a mismatched branch.
- [x] `App\Services\CreditCards` owns billing and financial behavior; DTOs carry multi-field create/move inputs. Direct model queries suffice; no new repository or global state.
- [x] Vue Composition API with project-standard JavaScript `<script setup>`; Axios service owns transport, Pinia owns cross-view card snapshots, Element Plus owns forms/selects/dialogs, Tailwind owns layout.
- [x] Backend feature/unit and contract coverage precedes frontend service/component coverage and isolated Playwright for create, both move modes, and delete.
- [x] Owner isolation, exact money, stale-preview handling, permanent-delete replay safety, and no new secrets/packages/uploads are documented in the research, model, and contract.

## Design

### Backend sequence

1. Add `statement_closing_date` to manual create only. A statement-options query returns whether the date-derived cycle has closed and windowed choices, including an on-demand empty future cycle. Use closing date as cycle identity because empty cycles lack a database ID; resolve existing statement rows before synthesizing with current card billing days.
2. Add read-only move preview and idempotent move mutation. Both modes validate same-card eligibility and unchanged purchase fields. Full mode rebuilds consecutive cycles; one mode retains other allocations and enforces nondecreasing statement dates. Under card/purchase/statement locks, compare preview state, move existing installment rows, record move history, and reconcile credit-event applications.
3. Add confirmed, idempotent hard delete. Reject any linked refund/correction credit; remove purchase, installments, and move history. Preserve payments and account debits. Mark a linked recurring occurrence as purchase-deleted while keeping it non-generatable. Scrub cached purchase response snapshots without dropping idempotency keys; return minimal delete summary.
4. Extend card-wide obligation reconciliation with a payment-surplus source and application ledger. Rebuild active credit allocations deterministically after financial mutations, apply available credit to oldest unpaid statements, derive used credit from per-statement outstanding, and capture affected budget/statement notifications. Keep historical credit allocations readable via supersession rather than silent overwrite.
5. Prove date boundaries, old billing-day snapshots, linked credits, partial/paid statements, payment updates/removal/restoration, repeated/concurrent requests, recurrence, and exact once dashboard/budget/history results before frontend starts.

### Frontend component map

| Component | Responsibility | Inputs | Outputs |
| --- | --- | --- | --- |
| `CreditCardPurchaseForm` | Create purchase and require explicit closed-cycle statement choice | card, visible, submitting, statement options, field errors | submit, close, options request |
| `CreditCardPurchaseMoveDialog` | Choose mode, installment, target, preview, and confirm | purchase, options, preview, loading, errors | preview request, confirm, close |
| `CreditCardPurchaseDeleteDialog` | Explain permanent removal and confirm | purchase, linked-credit eligibility, loading, error | confirm, close |
| `CreditCardPurchaseList` / `ExpandableTransactionItem` | Expose owned purchase actions, passing installment context from statement view | purchase, installment, loading | move, delete |
| `CreditCardDetailView` / `CreditCardStatementView` | Compose dialogs, fetch full purchase when needed, refresh route data | route card/statement IDs | feature action orchestration |

Keep billing calculations, surplus, and statement totals on the server. Date/card changes clear an explicit selection; stale options show field errors. Move preview and Delete confirmation keep focus within Element Plus dialogs, restore focus on close, and show durable refreshed content. Follow [design foundation](../../docs/design/design-foundation.md), [app shell](../../docs/design/app-shell.md), [navigation](../../docs/design/navigation.md), and [components](../../docs/design/components.md). Add pt-BR/en text, 44px targets, semantic tokens, and keyboard/200% zoom coverage.

## Project Structure

### Documentation (this feature)

```text
specs/024-manage-card-purchases/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── usability-results.md         # Recorded during final verification
├── contracts/purchase-statement-management-api.yaml
└── tasks.md
```

### Source Code (sibling repositories)

```text
../zunera-backend/
├── routes/api.php
├── app/Http/{Controllers/Api/V1,Requests/CreditCards,Resources/CreditCards}/
├── app/Data/CreditCards/
├── app/Services/CreditCards/
├── app/Services/RecurringTransactions/
├── app/Models/
├── database/migrations/
└── tests/{Feature,Unit}/CreditCards/

../zunera-frontend/
├── src/components/credit-cards/
├── src/views/credit-cards/
├── src/services/creditCardService.js
├── src/stores/credit-cards/creditCardStore.js
├── src/i18n/messages.js
├── src/**/__tests__/
└── e2e/{credit-cards.spec.js,support/creditCardFixtures.js}
```

**Structure Decision**: Extend existing card feature folders. Add focused move/delete services and dialogs. Keep account settlement and recurrence integrations in their existing owners; do not refactor unrelated card screens.

## Phase Outputs and Gates

- **Phase 0**: [research.md](research.md) records decisions, rationale, alternatives, and existing financial constraints.
- **Phase 1**: [data-model.md](data-model.md), [API contract](contracts/purchase-statement-management-api.yaml), and [quickstart.md](quickstart.md) define persistence, request/response rules, and verification journeys.
- **Phase 2**: `/speckit-tasks` will produce dependency-ordered tasks with backend-first gates. Implementation and application tests are outside this planning command.

## Complexity Tracking

No constitution exceptions. The payment-surplus ledger and move history add persistence because existing credit-event applications cannot represent retained payments after permanent purchase deletion or preserve traceable moves.
