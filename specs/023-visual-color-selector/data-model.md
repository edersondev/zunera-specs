# Data Model: Visual Color Selector

The existing `color` string field on financial accounts, categories, and credit cards remains unchanged. Values are one of `teal`, `blue`, `indigo`, `violet`, `purple`, `pink`, `rose`, `red`, `orange`, `amber`, `lime`, `green`, `emerald`, `cyan`, `sky`, `slate`. All fit existing 24/32-character columns. Null or absent input retains current service defaults (`teal` for accounts/categories; `violet` for cards). Existing category color snapshots are strings and need no migration.
