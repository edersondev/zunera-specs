# Form Page

## Wide Layout

```text
┌──────────────┬──────────────────────────────────────────────────┐
│ Navigation   │ Create record                    [Cancel]         │
│              │ Add required information below.                  │
│              │                                                   │
│              │ Details                                           │
│              │ Name *                                            │
│              │ [______________________________________________] │
│              │                                                   │
│              │ Status                                            │
│              │ [Select status                                ▾] │
│              │                                                   │
│              │ Notes                                             │
│              │ [______________________________________________] │
│              │ [______________________________________________] │
│              │                                                   │
│              │                    [Cancel] [Create record]     │
└──────────────┴──────────────────────────────────────────────────┘
```

Use `<ElForm label-position="top">` and `<ElFormItem>` with visible labels,
required indicators, help text, and inline validation. On compact screens,
fields and actions use a single column; primary action remains last in visual
order. Submit loading state prevents repeats. Success routes to durable updated
content; server errors map to their affected fields or a page-level alert.
