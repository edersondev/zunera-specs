# Feature Specification: Visual Color Selector

**Feature Branch**: `023-visual-color-selector`  
**Backend Branch**: `023-visual-color-selector` (`../zunera-backend`)  
**Frontend Branch**: `023-visual-color-selector` (`../zunera-frontend`)  
**Created**: 2026-10-07  
**Status**: Ready

## User Scenarios & Testing

### User Story 1 - Choose a color visually (Priority: P1)

While creating or editing a financial account, category, or credit card, a user sees the current color and its name in the closed field and chooses from the same compact palette in all three forms.

**Independent Test**: Open each form, select a different swatch, save, and reopen. The name, swatch, and persisted value match the choice.

**Acceptance Scenarios**:

1. **Given** a form with a selected color, **when** the form opens, **then** the Color field shows its actual swatch, localized name, and chevron.
2. **Given** an open palette, **when** the user selects a swatch, **then** the closed field immediately reflects that color and the saved record retains it.

### User Story 2 - Operate by keyboard or assistive technology (Priority: P1)

A keyboard or screen-reader user can identify and choose any color, including the current one, without relying on hue alone.

**Independent Test**: Navigate the palette with keyboard alone and inspect accessible labels and selected state.

**Acceptance Scenarios**:

1. **Given** the Color field has focus, **when** it opens, **then** focus enters the palette at the selected option.
2. **Given** the palette is open, **when** the user navigates with arrow keys and chooses with Enter or Space, **then** the chosen option is applied and focus returns to the field.
3. **Given** the palette is open, **when** Escape is pressed, **then** it closes without changing the value and focus returns to the field.

### User Story 3 - Keep existing records correct (Priority: P1)

Previously saved colors remain recognizable after the new palette is released.

**Independent Test**: Load records with every previously accepted value and verify that its mapped swatch and localized name appear without changing its stored string.

**Acceptance Scenarios**:

1. **Given** a record with a deprecated color value, **when** it is viewed or edited, **then** the mapped swatch, localized name, and selected indicator appear.
2. **Given** an edit with no new color selection, **when** it is saved, **then** the API retains the original stored color string.
3. **Given** any of the ten selectable colors, **when** the record is saved and reloaded, **then** the chosen semantic string and visual color return.

### Edge Cases

- A narrow dialog or 320px viewport must keep the whole palette visible without horizontal overflow.
- Focus and hover names must remain available in both Portuguese and English.
- Unknown color values remain rejected by the API; null or missing color defaults to cyan for new accounts/categories and violet for new cards.

## Requirements

### Functional Requirements

- **FR-001**: All three Color fields MUST use one shared visual selector and one ordered palette of exactly 10 semantic values: blue, violet, pink, red, orange, yellow, green, cyan, brown, gray.
- **FR-002**: The closed field MUST match adjacent controls in size, typography, radius, and focus treatment, and show a 12–14px actual-color swatch, localized name, and chevron.
- **FR-003**: The open field MUST present a compact five-column/two-row swatch grid when space permits, with a clear selected indicator, selected color name, and localized names on hover and focus.
- **FR-004**: Every option MUST have an accessible name, keyboard access, visible focus, and a non-color selected indicator. The selector MUST expose expanded and selected states.
- **FR-005**: The palette MUST adapt to narrow dialogs and 320px screens, respect both themes and reduced motion, and avoid layout overflow.
- **FR-006**: Stored values and request/response field shapes MUST remain unchanged. The API MUST accept all 16 previously supported values plus yellow, brown, and gray. New accounts/categories default to cyan; cards default to violet.
- **FR-007**: Other form fields and layouts MUST remain unchanged.
- **FR-008**: Presentation-only aliases MUST map indigo→blue, purple→violet, amber→orange, lime→green, emerald→green, sky→cyan, teal→cyan, rose→pink, and slate→gray. New selections emit only the ten current values.

### Security and Quality Requirements

- **SQR-001**: Existing authentication and authorization remain; backend request validation accepts only the ten current or nine legacy values.
- **SQR-002**: Backend contract tests cover current, legacy, default, and invalid values. Component tests cover accessibility and interaction; Playwright covers the three critical save flows.
- **SQR-003**: No new secrets, uploads, packages, or database migration.

### Delivery Scope

1. **Backend** (`../zunera-backend`): Extend the allowed semantic color values consistently for financial accounts, categories, and credit cards; update new-record defaults while preserving authorization, stored values, and resource output; add tests.
2. **Frontend** (`../zunera-frontend`): Shared selector, centralized palette and translations, theme tokens, integration into the three forms, tests and design documentation, after backend contract passes.

### Key Entities

- **Visual color**: A stable semantic string saved with a financial account, category, or credit card and mapped to a localized name and theme-aware swatch.

## Success Criteria

### Measurable Outcomes

- **SC-001**: All three forms offer the same 10 colors in a 5 × 2 grid on normal widths and save every new choice correctly.
- **SC-002**: All nine deprecated values display a current equivalent without rewriting the persisted value on an unchanged edit.
- **SC-003**: Every choice can be made without a mouse, with a visible focus and selected indication.
- **SC-004**: No horizontal overflow occurs in the Color palette at 320px viewport width or 200% zoom.

## Assumptions

- `pink` is “Rosa” in pt-BR; legacy `rose` is presented as pink. Legacy `teal`, `rose`, and `slate` map to cyan, pink, and gray respectively.
- Existing API field name and semantic string format remain stable. Clients may still send previously accepted values.
- Backend support is delivered before the frontend offers new colors.
