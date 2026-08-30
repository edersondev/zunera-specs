# Dashboard

## Wide Layout

```text
┌──────────────┬──────────────────────────────────────────────────┐
│ ZUNERA       │ Workspace ▾                     Notifications  ◉ │
│              ├──────────────────────────────────────────────────┤
│ Dashboard  ● │ Dashboard                                        │
│ Records      │ Welcome back, [User]                             │
│ Reports      │                                                   │
│ Settings     │ [Metric]        [Metric]        [Metric]          │
│              │                                                   │
│              │ Recent activity                 Quick actions     │
│              │ ┌────────────────────────────┐  [Create record]  │
│              │ │ activity list               │  [View reports]  │
│              │ └────────────────────────────┘                   │
└──────────────┴──────────────────────────────────────────────────┘
```

On compact viewports, primary navigation moves to drawer and all dashboard
regions stack into one column. Each metric has descriptive label and status;
recent activity has loading, empty, and error states. Quick actions use visible
text labels and respect user permissions.
