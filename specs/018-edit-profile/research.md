# Research: Edit Profile

## Existing authentication

- Registration trims name and normalizes email. Profile editing reuses name constraints; email remains read-only.
- The current auth session response already includes the user fields needed to prefill the dialog. `UserResource` remains the response serializer.
- Password reset already defines safe password rules, password-change notice, recovery tokens, and database-session revocation. Signed-in password change reuses these patterns while preserving the initiating session.
- Shared current-password failure limit is scoped to signed-in user ID to avoid bypass across repeated attempts. No new package or table is needed.

## UI boundary

- The account menu emits Edit profile and Change password actions. App shell owns both dialog visibility states; each focused dialog owns one form. Auth service performs requests and the shared session store updates header identity.
- Email uses a disabled, labeled field and is omitted from the profile request.
