# Color API Contract

Existing authenticated create/update endpoints for financial accounts, categories, and credit cards keep the `color` request and response field as a semantic string. Accepted non-null values are `teal`, `blue`, `indigo`, `violet`, `purple`, `pink`, `rose`, `red`, `orange`, `amber`, `lime`, `green`, `emerald`, `cyan`, `sky`, and `slate`. Unsupported values return existing validation errors. Existing defaults, authorization, response shapes, and stored values remain unchanged.
