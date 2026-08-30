# Navigation

## Purpose

Navigation tells users where they are, what they can do next, and how to return
without losing context. It MUST reflect authorization and current workspace.

## Levels

| Level | Purpose | Element Plus pattern |
| --- | --- | --- |
| Primary | Main application areas | `el-menu` in persistent aside or drawer |
| Contextual | Sections within current area | `el-menu`, tabs, or secondary list |
| Local | Content anchors or controls | Tabs, segmented controls, or in-page links |
| Path | Hierarchy back to parent resources | `el-breadcrumb` |

Use one primary navigation region. Contextual navigation appears only when it
reduces complexity; do not duplicate primary destinations as tabs.

## Primary Navigation Model

Each item contains stable route name, visible label, icon when it improves
recognition, permission requirement, and optional badge count. Use route names
instead of literal URLs for active-state and permission logic.

```ts
type NavigationItem = {
  routeName: string
  label: string
  icon?: Component
  permission?: string
  badgeCount?: number
}
```

Build visible items in composable or store selector. `AppNavigation` receives
items as read-only props and emits navigation intent; it does not call API
services or mutate authorization state.

## Active and Disabled States

- Active item uses Element Plus active state plus `aria-current="page"`.
- Parent item remains identifiable when child route is active.
- Hidden items represent unavailable permissions; disabled items represent a
  visible, temporarily unavailable action and MUST explain why.
- Badge counts are supplemental information, never sole indication of urgency.

## Responsive Behavior

On wide viewports, primary navigation remains in left aside. On compact
viewports, same items render in `el-drawer`; item order, labels, permissions,
and active state remain identical. Drawer trigger has accessible name such as
"Open navigation". Escape, close button, overlay click when safe, and route
selection close drawer and restore focus to trigger or new page heading.

## Tabs and Breadcrumbs

Use tabs for peer views sharing one resource context, such as overview, activity,
and settings. Each tab has concise label and preserves URL state. Do not use tabs
for sequential process; use steps or explicit actions instead.

Use breadcrumbs only when parent-child hierarchy aids recovery. Breadcrumbs do
not replace page title. Current page is plain text or has `aria-current="page"`.

## Navigation Feedback

- Route change updates document title and moves focus to `h1` or main region.
- Loading transition preserves current navigation and announces progress when
  loading exceeds brief delay.
- Failed navigation shows recoverable error in alert region, retaining previous
  stable route when possible.
- Unsaved changes require confirmation before navigation discards data.

## Accessibility and Testing

- `el-menu` and drawer keyboard behavior MUST remain intact; do not override
  focus styles.
- Icons require visible text or accessible label.
- Target size is at least 44 × 44px on touch layouts.
- Playwright covers primary route change, active state, compact drawer, keyboard
  close, and authorization-hidden item for critical areas.
