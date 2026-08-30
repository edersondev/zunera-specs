# List Page

## Wide Layout

```text
┌──────────────┬──────────────────────────────────────────────────┐
│ Navigation   │ Records                           [+ New record] │
│              │ Search [________________]  [Status ▾] [Reset]    │
│              │                                                   │
│              │ ┌──────────────────────────────────────────────┐ │
│              │ │ Name          Status       Updated     Actions│ │
│              │ │ Record A      Active       Today       View ⋮ │ │
│              │ │ Record B      Draft        Yesterday   View ⋮ │ │
│              │ └──────────────────────────────────────────────┘ │
│              │ Showing 1–20 of 120                 ‹ 1 2 3 ›   │
└──────────────┴──────────────────────────────────────────────────┘
```

Filter values, sorting, and pagination persist in route state. On compact
screens, filters stack and table preserves critical columns through responsive
priority, horizontal overflow, or a detail view. Loading, empty, and error
states replace table body without removing page title, filters, or retry action.
