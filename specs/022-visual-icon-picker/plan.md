# Implementation Plan: Visual Icon Picker

**Branch**: `022-visual-icon-picker` | **Spec**: [spec.md](spec.md)

## Backend scope

1. Extend the existing Financial Accounts, Categories, and Credit Cards icon allowlists. Keep stored values, string payloads, endpoint authorization, and validation rules otherwise unchanged.
2. Test new, legacy, and invalid values on create and update before starting frontend work.

## Frontend scope

1. Add a shared icon registry, localized labels, and curated lists for accounts, categories, and cards.
2. Build a reusable `IconPicker` using Element Plus Popover, then use it in all three forms and existing icon displays.
3. Test keyboard, search, saved values, dialog positioning, themes, and responsive behavior.

## Contract and design

- API `icon` remains a string; defaults and stored values remain unchanged. No migration or new package.
- New values: `cash`, `travel`, `subscription`, `utilities`. Accounts add `credit_card`, `cash`, `briefcase`; Categories add `bank`, `piggy_bank`, `credit_card`, `smartphone`, `cash`, `travel`, `subscription`, `utilities`; Cards add `shopping_bag`, `travel`, `subscription`, `gift`.
- The shared registry owns icon components and context membership; existing context-specific i18n labels remain where their meaning differs.
- Popover stays anchored inside modal focus scope, flips or shifts within viewport, and scrolls only its icon grid. Tiles use semantic theme tokens and three or four columns based on available width.
- Keyboard uses focusable options, roving focus, arrow navigation, Enter/Space selection, Escape close, and focus return. Category search filters localized labels without case or accent sensitivity.

## Constitution Check

- **Before research:** Reuse the existing protected API and `icon` string contract. Keep request validation and domain services in their current boundaries, avoid a migration or new package, and complete backend contract tests before frontend integration.
- **After design:** The frontend registry and backend allowlists contain matching values for each context. The shared Vue component owns only picker state; forms retain their current API and state boundaries. Contract, unit, and browser tests cover saved values, validation, keyboard behavior, and responsive layouts. The changes stay within the three icon fields and their displays.
- **Complexity Tracking:** No constitutional exceptions or new dependencies.

## Verification

- Backend: run `php artisan test --filter=IconValuesContractTest` and the full `php artisan test` suite in the live backend container, then run Pint.
- Frontend: run `npm run test:unit -- --run`, `npm run lint`, and the three affected Playwright specs after building the frontend.
- Record any unrelated suite failure separately from icon-picker results.
