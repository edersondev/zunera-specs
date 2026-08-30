# Login

## Compact and Wide Layout

```text
┌──────────────────────────────────────────────┐
│ ZUNERA                                       │
│                                              │
│ Welcome back                                 │
│ Sign in to continue to your workspace.       │
│                                              │
│ Email address                                │
│ [__________________________________________] │
│                                              │
│ Password                                     │
│ [__________________________________________] │
│                                              │
│                         Forgot password?     │
│                                              │
│ [               Sign in                    ] │
│                                              │
│ Need help? Contact support.                  │
└──────────────────────────────────────────────┘
```

Use one centered surface on `--color-canvas`. Visible labels remain above
fields; validation appears inline beneath its related field. Submit shows a
loading state and prevents duplicate submission. On failed sign-in, show a
page-level alert above fields while preserving entered email address.
