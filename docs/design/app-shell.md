# App Shell

## Purpose

App shell is persistent frame around authenticated Zunera views. It provides
identity, workspace context, primary navigation, global feedback, and safe
content area. Route views compose feature components; they MUST NOT reimplement
shell behavior.

## Structure

```text
AppShell
├── SkipLink
├── AppHeader
│   ├── Brand
│   ├── WorkspaceSwitcher
│   ├── GlobalActions
│   └── AccountMenu
├── AppNavigation
│   ├── PrimaryNavigation
│   └── ContextNavigation (when route requires it)
├── MainContent
│   ├── PageHeader
│   ├── PageActions
│   └── RouteView
└── GlobalFeedback
```

Use semantic landmarks in this order: skip link, `header`, `nav`, `main`, then
optional `aside`. `main` owns focus after each route transition.

## Layout Behavior

| Range | Navigation | Content |
| --- | --- | --- |
| Compact, under 640px | Element Plus drawer | Single column; 16px gutters |
| Medium, 640px–1023px | Drawer or rail | Single or two columns; 24px gutters |
| Wide, 1024px+ | Persistent left navigation | Centered workspace; 32px gutters |

Header height is 64px. Wide navigation is 256px; compact touch controls remain
at least 44px square. Shell does not impose fixed content height; route views
own vertical content and scrolling.

## Vue Component Map

| Component | Responsibility | Inputs | Outputs |
| --- | --- | --- | --- |
| `AppShell` | Composes persistent frame and route slot | route metadata | none |
| `AppHeader` | Shows brand, workspace, and account controls | workspace, user | `change-workspace`, `open-menu` |
| `AppNavigation` | Renders current navigation model | items, current route | `navigate` |
| `PageHeader` | Renders title, context, and page actions | title, description, actions | action events |
| `GlobalFeedback` | Hosts alerts and transient feedback | feedback queue | dismiss events |

Use typed `defineProps` and `defineEmits`. Props flow down and events flow up.
Shell state that crosses route boundaries belongs in focused Pinia store;
route-local state belongs in its view or feature composable.

## Required States

- **Loading**: Preserve shell; show page skeleton or Element Plus loading state
  inside `main`.
- **Empty**: Render contextual empty state with one clear next action.
- **Error**: Keep navigation available; put page-level failure in alert region.
- **Unauthorized**: Use dedicated route state; do not render protected content
  beneath a message.
- **Offline or retrying**: Describe affected action and offer retry when safe.

## Page Header

Every route has one `h1`. Page header MAY include breadcrumb, short description,
status, and actions. Primary action appears last in visual order on wide screens.
Secondary actions use Element Plus button variants or dropdown; destructive
actions never share primary styling.

## Implementation Rules

- Use `el-container`, `el-header`, `el-aside`, and `el-main` only when behavior
  fits layout; Tailwind owns responsive composition and gutters.
- Use `el-drawer` for compact navigation. Opening drawer moves focus inside;
  closing returns focus to trigger.
- Derive active route and visible navigation with `computed`; watchers handle
  effects such as focus restoration only.
- Keep `AppShell.vue` thin. Extract header, navigation, and feedback into
  separate components.
- Apply theme tokens from `design-foundation.md`; no per-route raw colors.

## Acceptance Checks

- Keyboard user can skip directly to `main` and reach all shell actions.
- Current route has visible and programmatic navigation state.
- Compact navigation opens, closes, and restores focus correctly.
- Route transition leaves no stale loading or feedback state.
- Shell is usable at 200% zoom and from 320px viewport width.
