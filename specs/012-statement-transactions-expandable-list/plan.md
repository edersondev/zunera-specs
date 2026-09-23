# Implementation Plan: Expandable Statement Transactions

**Branch**: `012-statement-transactions-expandable-list` | **Date**: 2026-09-23 | **Spec**: [spec.md](spec.md)

**Branch Coordination**: The same feature branch is active in specs, backend and frontend, based on `main`.

## Summary

Expose purchase identity and presentation metadata on existing statement installments, then replace the flat installment rows with compact expandable items. Existing correction/refund dialogs and mutations are reused; no financial calculation changes.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13; JavaScript/Vue 3.5
**Primary Dependencies**: Laravel API Resources, Vue Composition API, Pinia, Axios, Element Plus, Tailwind 4, Vue I18n
**Storage**: Existing credit-card tables; no migration
**Testing**: PHPUnit, Pint, Vitest, Playwright, ESLint, Vite build
**Target Platform**: Authenticated Zunera web app, light/dark, 320px and larger
**Project Type**: Full-stack response enrichment and focused UI feature
**Performance Goals**: No extra request per rendered row; purchase details fetched only on action
**Constraints**: Exact centavos, unchanged statuses and allocation, no new package
**Scale/Scope**: One statement's installments; no page-wide redesign

## Delivery Scope and Order

1. **Backend** — Add read-only purchase metadata to the existing owned statement response and test its shape and unchanged financial values.
2. **Frontend** — Consume verified fields in an accessible expandable list; reuse existing purchase dialogs and action APIs; test and validate responsive presentation.

## Constitution Check

*GATE: Passed before research; re-checked after design: Passed.*

- [x] Existing Sanctum route and owner scope remain; API Resource supplies read-only fields; no new input or upload.
- [x] Backend contract and tests precede frontend use.
- [x] All three repositories use the same branch.
- [x] Financial behavior remains in existing services; no new DTO or repository.
- [x] Vue components use `<script setup>`, explicit props/emits, feature store and service for API, Element Plus controls and semantic tokens.
- [x] Backend contract, frontend interaction and critical Playwright action flows are covered.
- [x] No secrets, migration or package change.

## Design

- Statement installment response adds `purchase_id`, `category` (`id`, `name`, `icon`, `color`), `purchase_total_amount`, and `is_directly_editable`. Existing fields remain unchanged. Category metadata comes from the purchase's owned relationship. Only this statement response needs the additive fields.
- `CreditCardStatementLineItems` owns the single expanded installment ID and clears it when the list changes. `ExpandableTransactionItem` presents one installment and emits toggle/correct/refund intent; it performs no calculations or requests.
- The statement view owns existing dialogs and uses `getPurchase` through the credit-card store when an action opens. Correct is shown only when eligible; refund uses existing server eligibility. Success refreshes the statement and current card activity; errors use existing feedback.
- The collapsed amount is the existing installment `amount`. Expanded adjustment and `recognized_amount` are separately labeled when credit adjustment is nonzero. Payments and credit events remain separate sections.
- Reuse `formatBRL`, `formatIsoDate`, `formatStatementMonth`, `cardIdentityLabel`, `installmentLabel`, `recognitionStatus`, category icon mapping, existing theme CSS, and Portuguese/English messages. Focus uses global 2px visible ring; reduced motion uses global setting.

## Project Structure

```text
specs/012-statement-transactions-expandable-list/{spec,plan,research,data-model,quickstart,tasks}.md
specs/012-statement-transactions-expandable-list/contracts/statement-installment-response.yaml
../zunera-backend/app/Http/Resources/CreditCards/CreditCardStatementDetailResource.php
../zunera-backend/tests/Feature/CreditCards/CreditCardStatementTest.php
../zunera-frontend/src/{components/credit-cards,views/credit-cards,stores/credit-cards,i18n}/
../zunera-frontend/e2e/credit-cards.spec.js
```

## Complexity Tracking

No constitution exception.
