# Zunera Design Foundation

## Purpose and Scope

This document is the source of truth for Zunera's visual language, global
interaction rules, and semantic design tokens. It ensures financial information
is clear, interfaces are accessible, and features remain coherent as the product
grows.

Every frontend feature MUST follow this foundation. Feature-level decisions MAY
extend it when needed, but MUST NOT contradict it. Application layout,
navigation, reusable components, and screen structures are defined respectively
in `app-shell.md`, `navigation.md`, `components.md`, and `wireframes/`.

## Design Principles

- **Simple:** Prioritize clear tasks and information. Avoid decorative clutter,
  redundant content, and unnecessarily complex navigation.
- **Financially clear:** Balances, income, expenses, budgets, and totals have a
  strong visual hierarchy and are easy to compare.
- **Consistent:** Equivalent actions, data, and states look and behave the same.
  Reuse documented Element Plus and Zunera components before adding new patterns.
- **Accessible:** Keyboard operation, visible focus, semantics, labels, and
  sufficient contrast are baseline requirements.
- **Responsive:** Layout adapts to available space; mobile is not a compressed
  desktop view.
- **Theme-aware:** Components use semantic tokens, never assumptions such as
  “white card” or “gray border”.

Color MUST NOT be sole source of financial or status meaning. Pair it with
clear labels, values, icons, signs, or supporting text.

```text
+ R$ 5.000,00  Income
- R$ 2.350,00  Expense
```

## Technology Ownership

| Concern | Standard |
| --- | --- |
| Application UI | Vue 3 Composition API and JavaScript `<script setup>` |
| Component controls | Element Plus, preferred for standard controls |
| Layout and composition | Tailwind CSS v4 utilities |
| Shared client state | Pinia |
| API access | Axios services |
| End-to-end journeys | Playwright |

Use Element Plus for standard forms, buttons, dialogs, tables, feedback, and
pagination. Use Tailwind for responsive layout, spacing, and small
presentational composition. Do not recreate an Element Plus control with custom
markup unless a documented requirement cannot be met by the library.

## Visual Direction and Semantic Meaning

Zunera is modern, calm personal-finance application. Teal communicates brand
and primary actions. Slate-based neutrals provide structure. Green and red are
reserved for financial and status meaning, not decoration.

```text
Teal    → brand, primary action, active selection
Green   → positive result, income, success
Red     → negative result, expense, error, destructive action
Amber   → warning or attention needed
Blue    → neutral information
Neutral → surfaces, structure, standard information
```

The current balance MUST use the standard text color unless its positive or negative
state is itself meaningful. Do not color balance green solely because it is
greater than zero. Use tabular figures (`font-variant-numeric: tabular-nums`)
for aligned financial values where the selected font supports them.

## Theme Preferences

Zunera supports `light`, `dark`, and `system` preferences. `system` SHOULD be
the default for new users, and an explicit user choice SHOULD be persisted.

Components consume semantic tokens only. Active theme maps those tokens to
actual values:

```text
Component → semantic token → light or dark value
```

Components MUST NOT use raw theme colors such as `#FFFFFF`, `white`, or
`gray-200` in templates. Raw values belong only in global theme definitions.

## Color Tokens

Use namespaced CSS custom properties in implementation. Element Plus variables
MUST map to these semantic tokens globally.

### Brand and Neutral Tokens

| Token | Light | Dark | Use |
| --- | --- | --- | --- |
| `--color-canvas` | `#F8FAFC` | `#0B1120` | Application background |
| `--color-surface` | `#FFFFFF` | `#111827` | Cards, panels, dialogs |
| `--color-surface-secondary` | `#F1F5F9` | `#1F2937` | Grouped or muted surface |
| `--color-surface-tertiary` | `#E2E8F0` | `#374151` | Stronger neutral surface |
| `--color-border` | `#E2E8F0` | `#374151` | Standard boundaries |
| `--color-border-strong` | `#CBD5E1` | `#4B5563` | Emphasized boundaries |
| `--color-text` | `#0F172A` | `#F8FAFC` | Primary text |
| `--color-text-muted` | `#64748B` | `#94A3B8` | Supporting text |
| `--color-text-subtle` | `#475569` | `#CBD5E1` | Readable secondary text |
| `--color-text-disabled` | `#94A3B8` | `#64748B` | Disabled content |
| `--color-action-primary` | `#0F766E` | `#2DD4BF` | Primary action and selection |
| `--color-action-primary-hover` | `#115E59` | `#5EEAD4` | Hover state |
| `--color-action-primary-active` | `#134E4A` | `#14B8A6` | Active state |
| `--color-action-primary-subtle` | `#CCFBF1` | `#134E4A` | Subtle selected state |
| `--color-on-primary` | `#FFFFFF` | `#042F2E` | Content on primary surface |

### Status and Financial Tokens

| Token | Light | Dark | Use |
| --- | --- | --- | --- |
| `--color-success` | `#16A34A` | `#4ADE80` | Successful operation or positive status |
| `--color-danger` | `#DC2626` | `#F87171` | Error or destructive action |
| `--color-warning` | `#D97706` | `#FBBF24` | Attention needed or approaching limit |
| `--color-info` | `#2563EB` | `#60A5FA` | Neutral information |
| `--color-financial-positive` | `#16A34A` | `#4ADE80` | Income and positive movement |
| `--color-financial-negative` | `#DC2626` | `#F87171` | Expense and negative movement |

Status colors MUST be paired with text, icon, or label. Verify foreground and
background combinations against WCAG 2.2 AA; semantic colors may require a
different shade when used as small text on a surface.

### Chart Palette

Use this categorical palette only when category distinction is needed:

| Teal | Blue | Violet | Amber | Rose | Cyan |
| --- | --- | --- | --- | --- | --- |
| `#14B8A6` | `#3B82F6` | `#8B5CF6` | `#F59E0B` | `#F43F5E` | `#06B6D4` |

Category color MUST NOT imply financial gain or loss. For direct income versus
expense comparisons, use financial positive and negative tokens. Charts MUST
provide labels, legends, values, patterns, or other non-color cues when meaning
would otherwise be ambiguous.

## Typography, Spacing, Shape, and Elevation

Use one system sans-serif stack until brand typography is approved. Maintain one
`h1` per page and follow semantic heading order. Financial values SHOULD use
tabular figures and avoid excessive size or weight variation.

| Role | Size / line-height | Weight | Use |
| --- | --- | --- | --- |
| Display | 30px / 36px | 700 | Page title only |
| Heading | 24px / 32px | 700 | Major section |
| Subheading | 20px / 28px | 600 | Card or panel title |
| Body | 16px / 24px | 400 | Default content |
| Body compact | 14px / 20px | 400 | Tables and support text |
| Label | 14px / 20px | 600 | Form labels and actions |
| Caption | 12px / 16px | 400 | Metadata only |

Base spacing is 4px. Use the scale below; arbitrary spacing SHOULD be avoided.

| Token | Value | Token | Value |
| --- | ---: | --- | ---: |
| `space-1` | 4px | `space-2` | 8px |
| `space-3` | 12px | `space-4` | 16px |
| `space-5` | 20px | `space-6` | 24px |
| `space-8` | 32px | `space-10` | 40px |
| `space-12` | 48px | `space-16` | 64px |

| Token | Value | Typical use |
| --- | --- | --- |
| `--radius-sm` | 4px | Compact controls |
| `--radius-md` | 8px | Inputs and buttons |
| `--radius-lg` | 12px | Cards |
| `--radius-xl` | 16px | Dialogs |
| `--radius-full` | 9999px | Badges only |

Default component padding is 16px; cards MAY use 24px on wide screens. Prefer
surface contrast, borders, spacing, and type hierarchy over shadows. Use subtle
elevation for raised cards, moderate elevation for popovers, and highest
elevation only for dialogs and overlays. Dark theme SHOULD favor borders and
surface contrast over prominent shadows.

## Responsive Layout

Design mobile first. Content uses a centered container with 16px compact, 24px
medium, and 32px wide gutters.

| Range | Width | Expected behavior |
| --- | --- | --- |
| Compact | under 640px | One column; condensed navigation |
| Medium | 640px–1023px | One or two columns when content permits |
| Wide | 1024px–1439px | Persistent navigation and multi-column workspace |
| Large | 1440px and above | Wider workspace without unreadable line lengths |

Tables MUST retain task-critical columns. Use column priority, horizontal
overflow, or a detail view rather than unreadably narrow cells. Follow
`app-shell.md` for shell layout and `navigation.md` for navigation behavior.

## Interaction, Forms, and Application States

Every applicable control defines default, hover, focus-visible, active,
disabled, loading, selected, error, and success states. Hover MUST NOT be the only
indication of interactivity. Focus indicators MUST remain visible in both
themes, use a 2px contrasting ring, and not rely on color alone.

Interactive targets MUST be at least 44 × 44px where space permits. Motion is
subtle, functional, never delays an action, and respects reduced-motion
preferences.

Forms MUST use visible labels, clear required or optional meaning, keyboard
operation, nearby validation, and an explicit submit action. Implement every form
with Element Plus `<ElForm label-position="top">` by default. Placeholder text
MUST NOT substitute for labels. Format financial inputs according to Brazilian
conventions when applicable, for example `R$ 1.250,50`.

Any asynchronous page or component considers these applicable states:

| State | Required behavior |
| --- | --- |
| Loading | Preserve expected layout with skeleton or Element Plus loading state. |
| Empty | Explain successful absence of data and offer appropriate next action. |
| Success | Show durable updated content; toast is supplemental feedback only. |
| Error | Explain safely, avoid internals, and provide recovery when possible. |
| Disabled | Explain unavailable action when context requires it; disabled is not validation. |
| Unauthorized | Use dedicated state; never render protected content beneath message. |

## Accessibility Requirements

WCAG 2.2 Level AA is the Zunera accessibility target. At minimum, features MUST:

- Meet 4.5:1 text contrast and 3:1 large-text and UI-boundary contrast.
- Use semantic landmarks and headings.
- Support logical keyboard navigation and visible focus.
- Give every form field a visible label and understandable validation message.
- Give icon-only controls an accessible name.
- Keep focus inside open modals and return it to the trigger on close.
- Work at 200% zoom, at 320px viewport width, and with reduced motion.
- Use accessible Playwright locators, such as role and label, for critical flows.

## Token Implementation Rules

Global theme CSS owns raw token values and maps Element Plus variables to
semantic layer. Components consume semantic Tailwind utilities such as
`bg-canvas`, `text-text`, and `border-border`, or corresponding CSS variables.
They MUST NOT introduce raw colors or ad-hoc theme overrides in individual
views.

At minimum, global Element Plus theme maps these variables:

```css
:root {
  --el-color-primary: var(--color-action-primary);
  --el-color-success: var(--color-success);
  --el-color-warning: var(--color-warning);
  --el-color-danger: var(--color-danger);
  --el-color-info: var(--color-info);
  --el-border-color: var(--color-border);
  --el-text-color-primary: var(--color-text);
  --el-text-color-regular: var(--color-text-muted);
  --el-bg-color: var(--color-surface);
  --el-fill-color-blank: var(--color-surface);
  --el-border-radius-base: var(--radius-md);
}
```

## Feature and Design Governance

During specification, identify user experience, state, responsive, and
accessibility requirements. During planning, reuse existing patterns and
justify a new shared component. During implementation, use semantic tokens and
validate both themes, responsive layouts, keyboard flow, and critical journeys.

When a feature needs a reusable pattern, evaluate existing components first. If
none fits, decide whether it is feature-local or system-wide. Document a new
system-wide pattern in this foundation or `components.md` before broad reuse.
Do not add a global visual convention merely for local convenience.
