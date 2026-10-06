# Visual Icon Picker

## Goal

Make icon selection visual and reusable in Financial Accounts, Categories, and Credit Cards while preserving stored icon strings, form payloads, and Zunera's light and dark themes.

## User stories

- As a user, I can recognize each icon in a compact grid before choosing it.
- As a keyboard user, I can open the picker, move through icons, select one, and close it without losing focus.
- As a user editing an existing record, I see its saved icon selected and can save without changing it.

## Requirements

- One shared icon registry maps every existing value and new financial values to Element Plus icon components and localized names.
- Each feature presents a relevant subset of the shared registry. Existing values remain valid and unchanged.
- The closed field shows icon, name, and chevron with form-control dimensions. The open panel uses a responsive grid, teal selected state, visible focus, viewport-safe placement, and a scrollable icon area.
- Categories offer localized, accent-insensitive search because their icon set is larger.
- Input remains keyboard accessible with Enter, Space, arrows, Escape, and logical focus handling inside dialogs.
- Backend validation accepts newly offered values and continues rejecting unrecognized values.

## Backend scope

- Keep the existing authorized create, update, and read endpoints and the string `icon` field unchanged.
- Extend each feature's validated icon allowlist without removing previously accepted values; cover create, update, readback, and invalid-value rejection with contract tests.

## Frontend scope

- Share one localized icon registry and picker component across the three forms, while keeping context-specific icon choices and current form payloads.
- Preserve saved-value previews in the forms and existing icon displays. Keep the picker usable inside dialogs with keyboard, pointer, search, and narrow-viewport interactions.

## Edge cases

- Editing a record with any previously accepted icon preserves that value unless the user selects another icon. An unrecognized stored value displays a safe fallback icon and its raw value rather than crashing.
- An empty category search shows the full list; a search with no matches shows an empty state. Search ignores accents and letter case.
- Escape closes the panel and returns focus to its trigger. Moving focus outside closes it without changing the value. The panel flips or shifts when space is limited and its icon grid scrolls within the available height.

## Security implications

- No endpoint, authorization rule, or data shape changes. Existing route protection remains in force.
- Backend validation rejects icon strings outside the feature's allowlist; the client registry is only a presentation aid, not an authorization or validation boundary.
- Labels come from static translations and user-entered search text is used only for local filtering.

## Acceptance

- Create and edit flows in all three areas submit the same `icon` string field as before.
- Old records reopen with the correct preview and appear correctly in existing icon displays.
- The picker works at 320px, 200% zoom, and in light and dark themes without horizontal scrolling or viewport overflow.

## Measurable outcomes

- Contract tests confirm a legacy and a newly offered icon can be created, updated, and read back in each feature, while an unknown icon is rejected.
- Browser tests confirm pointer selection in all three forms and keyboard navigation, selection, Escape closing, and focus return inside a picker dialog.
- At 320px width and 200% zoom, the panel stays inside the viewport, uses three columns when needed, and scrolls its icon grid without horizontal page scrolling.
