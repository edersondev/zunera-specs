# Components

## Purpose

This catalog defines reusable UI building blocks and ownership boundaries.
Prefer Element Plus for standard controls, then compose with Tailwind layout and
small Zunera components. A custom component exists only when product behavior,
accessibility, or composition cannot be expressed cleanly with Element Plus.

## Component Tiers

| Tier | Owner | Examples | Rule |
| --- | --- | --- | --- |
| Foundation | Theme CSS | tokens, typography, focus ring | Global and semantic |
| Library | Element Plus | button, form, table, dialog, drawer | Prefer before custom work |
| Composition | Zunera UI | page header, filter bar, empty state | Combines library parts |
| Feature | Feature folder | enrollment form, roster table | Encapsulates domain UI |

## Standard Components

Use PascalCase for Element Plus components in Vue templates, such as
`<ElButton>`, `<ElForm>`, and `<ElTable>`. Imported component names MUST also
use PascalCase. Props and event handlers follow Vue conventions: camelCase in
script and kebab-case when bound in templates.

| Need | Preferred component | Required behavior |
| --- | --- | --- |
| Primary or secondary action | `el-button` | Text label; icon-only action has accessible label |
| Text, selection, or date input | `el-input`, `el-select`, `el-date-picker` | Visible label, help text, validation state |
| Form structure | `ElForm` and `ElFormItem` | `label-position="top"`, field validation, and submit feedback |
| Records | `el-table` | Loading, empty, error, responsive column plan |
| Confirmation | `el-dialog` or `el-popconfirm` | Explain outcome; focus return on close |
| Transient feedback | `ElMessage` or `ElNotification` | Supplemental, not sole critical feedback |
| Inline status | `el-tag`, `el-alert`, `el-badge` | Text and semantic color together |
| No data | `el-empty` | Explain state and offer next action |
| Paginated list | `el-pagination` | Preserve filter, sort, and route state |

## Button Conventions

Every user-visible **Cancel** action in a dialog or drawer MUST use the same
treatment: a close icon, the `danger` button variant, and a visible text label.
When the related save, confirmation, or lifecycle operation is in progress, the
Cancel button MUST be disabled. This convention also applies when Cancel only
closes a detail drawer and does not start a mutation.

## Composition Components

| Component | Responsibility | Contract |
| --- | --- | --- |
| `PageHeader` | Title, description, breadcrumb, page actions | Props in; action events out |
| `FilterBar` | Presents filters and reset action | Model props and `update:*` events |
| `DataTableState` | Loading, empty, error wrapper around table slot | State props; retry event |
| `ConfirmAction` | Standard destructive confirmation copy and dialog | Action config; confirm/cancel events |
| `StatusBadge` | Maps domain status to label, tag type, and icon | Status prop only |

Composition components MUST remain domain-light. Domain terminology and API calls
belong to feature components and services.

## Vue Contracts and State

- Use `<script setup>` with JavaScript, PascalCase filenames, and explicit
  `defineProps`/`defineEmits` contracts for public component boundaries.
- Props are read-only. Child components emit intent; containers perform API work.
- Use `defineModel` only for genuine two-way component contracts.
- Keep derived class names, visible records, and action availability in
  `computed`; use watchers only for side effects.
- Extract reusable stateful behavior into focused composables; pure formatting
  stays in utility modules.
- Feature views compose child components and stores; they MUST NOT become
  multi-section implementation containers.

## Forms

All application forms MUST use Element Plus `<ElForm label-position="top">`,
with fields wrapped in `<ElFormItem>`. A different label position requires a
documented feature-specific need.

Each field has label, clear required or optional meaning, validation message, and
stable name. Validate server-side error responses through feature service and map
them to `ElForm` fields. Submit control enters loading state once; prevent repeat
submission. On success, show durable updated content plus optional toast.

When a dialog containing a form closes, it MUST clear client-side validation and
server field errors before the next open. Reopening the dialog starts with a
clean validation state; previously entered values may be restored only when an
explicit edit flow requires them.

Do not use placeholder text as label. Do not disable submit merely to hide an
error; explain incomplete or unavailable actions.

## Data Display

Tables are for comparison across records. Cards are for scan-friendly summaries.
Use descriptions for key-value detail. Large datasets require server-backed
filtering, sorting, and pagination. Table actions use text or accessible
tooltips; destructive action requires confirmation.

## Styling Rules

- Use semantic Tailwind utilities from `design-foundation.md`.
- Use Element Plus theme variables globally. Component-scoped overrides require
  documented need and MUST use existing semantic tokens.
- Do not add raw hex values, arbitrary spacing, or one-off shadows in templates.
- Use `scoped` styles only for styles not represented by tokens, Element Plus,
  or Tailwind utilities.
- Respect default Element Plus keyboard and ARIA behavior.

## Component Readiness Checklist

- Has one responsibility and explicit props/emits contract.
- Covers loading, empty, error, disabled, and success states when applicable.
- Works by keyboard, exposes visible focus, and has accessible name.
- Uses semantic tokens and responsive layout.
- Includes unit or composable test; critical UI-to-API flow includes Playwright.
- Is documented here before reuse across features.
