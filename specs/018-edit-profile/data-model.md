# Data Model: Edit Profile

No migration or new entity.

- `users.name`: Updated after trim and validation (2–255 characters).
- `users.email`: Returned for display; never accepted for mutation by this feature.
- `users.password`: Replaced with a hash after current-password and new-password validation.
- `password_reset_tokens`: Delete this user's outstanding tokens after password change.
- `sessions`: Delete this user's rows except the current session ID after password change. Name update does not touch sessions.
