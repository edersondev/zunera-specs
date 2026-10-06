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

## Acceptance

- Create and edit flows in all three areas submit the same `icon` string field as before.
- Old records reopen with the correct preview and appear correctly in existing icon displays.
- The picker works at 320px, 200% zoom, and in light and dark themes without horizontal scrolling or viewport overflow.
