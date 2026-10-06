# Implementation Plan: Visual Icon Picker

**Branch**: `022-visual-icon-picker` | **Spec**: [spec.md](spec.md)

## Order

1. Extend backend icon allowlists and test new, old, and invalid values on create and update.
2. Add a shared frontend icon registry, localized labels, and curated lists for accounts, categories, and cards.
3. Build a reusable `IconPicker` using Element Plus Popover, then use it in all three forms and existing icon displays.
4. Test keyboard, search, saved values, dialog positioning, themes, and responsive behavior.

## Contract and design

- API `icon` remains a string; defaults and stored values remain unchanged. No migration or new package.
- New values: `cash`, `travel`, `subscription`, `utilities`. Accounts add `credit_card`, `cash`, `briefcase`; Categories add `bank`, `piggy_bank`, `credit_card`, `smartphone`, `cash`, `travel`, `subscription`, `utilities`; Cards add `shopping_bag`, `travel`, `subscription`, `gift`.
- The shared registry owns icon components and context membership; existing context-specific i18n labels remain where their meaning differs.
- Popover stays anchored inside modal focus scope, flips or shifts within viewport, and scrolls only its icon grid. Tiles use semantic theme tokens and three or four columns based on available width.
- Keyboard uses focusable options, roving focus, arrow navigation, Enter/Space selection, Escape close, and focus return. Category search filters localized labels without case or accent sensitivity.
