# Research: Financial Accounts

## Decisions

### User-owned account API

- **Decision**: Add versioned authenticated JSON routes for financial accounts:
  list active accounts, create, read details, update, archive, restore, list
  archived accounts, and read an active-account balance summary. Protect all
  routes with the existing Sanctum/session middleware and scope every query and
  action to the signed-in user.
- **Rationale**: The feature is entirely authenticated and user-owned. A checked
  OpenAPI contract lets the frontend start only after backend behavior is
  explicit and covered.
- **Alternatives considered**: A frontend-only local account list was rejected
  because the spec requires durable ownership, authorization, and future
  financial history. A single catch-all update route for lifecycle actions was
  rejected because archive/restore have distinct state rules and feedback.

### Financial account domain ownership

- **Decision**: Put create, update, archive, restore, current-balance, and summary
  rules in a focused financial-account service. Use DTOs for create/update
  payloads and enums for account type and lifecycle state. Use direct
  user-scoped model queries; do not add a repository for v1.
- **Rationale**: The constitution requires service-layer business rules and DTOs
  when inputs contain multiple fields. Direct user-scoped queries cover simple
  CRUD and lifecycle access without adding abstraction.
- **Alternatives considered**: Controller-owned rules were rejected because they
  would mix validation, authorization, and domain behavior. A repository was
  rejected because the planned queries are straightforward and fully scoped by
  user ownership.

### Monetary precision

- **Decision**: Store and exchange money as signed integer centavos with
  `currency_code` fixed to `BRL` for this feature. Cap per-account values at
  +/-999,999,999,999 centavos (`R$ 9.999.999.999,99`). The frontend accepts and
  displays Brazilian currency strings, but the API contract uses centavos for
  exact calculations.
- **Rationale**: Integer centavos avoid floating-point precision loss and are
  easy to sum for account and active-account balances. BRL-only scope matches the
  specification and avoids multi-currency rules that have not been defined.
- **Alternatives considered**: Decimal strings were rejected because every
  consumer would need stricter parsing rules. Floating-point numbers were
  rejected because they can lose cent-level precision. Multi-currency support was
  rejected as future scope. Unbounded product values were rejected because they
  make validation and acceptance testing unclear.

### Current balance before movement features

- **Decision**: Persist `initial_balance_centavos` and expose
  `current_balance_centavos`. Until movement features exist, current balance
  equals initial balance and `has_financial_movements` is false. Future movement
  features may change current balance through their own rules.
- **Rationale**: The spec states initial balance is not income and detailed
  transaction rules are future scope. Exposing the current balance now keeps the
  UI contract stable while avoiding a fake transaction ledger.
- **Alternatives considered**: Creating an initial-income movement was rejected
  because the spec explicitly forbids treating initial balance as ordinary
  income. Omitting current balance was rejected because users must see it.

### Lifecycle and deletion

- **Decision**: Use two persisted lifecycle states: `active` and `archived`.
  Archive and restore are explicit actions; no permanent deletion is exposed.
  Archived accounts remain readable through archived/detail views but are
  excluded from normal new-operation choices and active combined balance. Users
  may archive their last active account; the active list becomes empty and the
  active combined balance becomes zero.
- **Rationale**: Clarification chose one canonical inactive state and excluded
  permanent deletion. Separate lifecycle actions keep state conflicts testable.
- **Alternatives considered**: Separate inactive/closed states were rejected as
  unnecessary for this feature. Soft deletion was rejected as the user-facing
  lifecycle mechanism because it hides records from normal queries and conflicts
  with historical access. Requiring at least one active account was rejected
  because the feature should respect users who temporarily have no active
  accounts in Zunera.

### Initial balance correction

- **Decision**: Allow editing `initial_balance_centavos` only while
  `has_financial_movements` is false. Reject later changes with a stable
  state-conflict response.
- **Rationale**: Once movements exist, changing the starting amount rewrites
  history. Future adjustment features should model corrections explicitly.
- **Alternatives considered**: Always-editable initial balance was rejected
  because it would undermine historical balances. Never-editable initial balance
  was rejected because users need a correction window after creation.

### Account name uniqueness

- **Decision**: Enforce unique normalized names among each user's active
  accounts. Normalize names by trimming surrounding spaces, collapsing repeated
  internal spaces, and comparing without case or accent differences. Archived
  accounts may keep duplicate names, but restore/rename cannot create an active
  duplicate.
- **Rationale**: Users select accounts by name in future financial operations, so
  active duplicates increase operational mistakes. Allowing archived duplicates
  preserves history when users reuse names.
- **Alternatives considered**: Globally unique names were rejected because names
  are personal. Unique across all owned accounts was rejected because it blocks
  natural reuse after archiving. Fully duplicate active names were rejected
  because they harm selection clarity.

### Account name, financial institution, color, and icon

- **Decision**: Treat account name as the user's label for the account and keep
  it separate from optional user-entered financial institution text. Color and
  icon use predefined accessible choices with defaults.
- **Rationale**: User-defined labels support examples like "Conta principal" or
  "Nu salário" while institution text supports examples like "Nubank" or "Banco
  do Brasil". This also works for cash, wallets, multiple accounts at one
  institution, and accounts without an institution. Predefined visual choices
  keep theme contrast, icons, and validation stable across Light, Dark, and
  System themes.
- **Alternatives considered**: A managed institution catalog was rejected because
  freshness and governance are separate scope. Using institution as the account
  name was rejected because not every account has an institution and users may
  need multiple labels under one institution. Arbitrary colors/icons were
  rejected because they can create inaccessible or broken visual identifiers.

### Frontend state and component boundaries

- **Decision**: Add a setup-style Pinia account store for shared account list,
  summary, selected detail, loading/error state, and actions. Keep API calls in a
  financial-account service. Use route-level views as composition surfaces and
  feature components for account list, summary, form, detail, and archive/restore
  confirmation.
- **Rationale**: Vue and Pinia guidance favor Composition API, explicit
  props/events, computed derived state, service-owned API access, and focused
  components. The account list, detail, form, and lifecycle flows are large
  enough to split.
- **Alternatives considered**: Putting all logic in one view was rejected because
  it would mix API orchestration, form state, list rendering, and lifecycle UI.
  Component-local API calls were rejected by the constitution.

### Frontend UX and design foundation

- **Decision**: Use the authenticated app shell with a Financial Accounts primary
  navigation item, one `h1` per route, Element Plus forms/dialogs/tables or
  responsive card lists as appropriate, top-position labels, durable inline
  content updates, and supplemental messages. Format currency as Brazilian real
  for display and input.
- **Rationale**: The design foundation requires financially clear, accessible,
  theme-aware, responsive layouts and Element Plus standard controls. Financial
  values need tabular figures and must not rely on color alone.
- **Alternatives considered**: A standalone page outside the shell was rejected
  because this is an authenticated workspace feature. Toast-only feedback was
  rejected because critical outcomes must remain visible.

## References

- Laravel 13 documentation: Form Requests, middleware, API resources, and HTTP
  JSON tests: https://laravel.com/docs/13.x
- Vue documentation: Composition API, `<script setup>`, props/emits, and
  composables: https://vuejs.org
- Pinia documentation: setup stores and store testing:
  https://pinia.vuejs.org
- Zunera design foundation: `docs/design/design-foundation.md`
- Zunera app shell: `docs/design/app-shell.md`
- Zunera navigation: `docs/design/navigation.md`
- Zunera component guidance: `docs/design/components.md`
