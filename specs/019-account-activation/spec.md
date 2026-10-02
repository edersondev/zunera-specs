# Feature Specification: Account Activation

**Feature Branch**: `019-account-activation`
**Status**: Approved for implementation
**Input**: New accounts begin inactive. Email carries a link to activate the account before sign-in.

## User journeys

1. A valid registration creates an inactive account, does not sign in, and confirms that an activation email is on its way.
2. The email explains that the account is inactive, offers a clear activation button, states the 24-hour expiry, and gives guidance if registration was unexpected.
3. Opening a valid, unused link activates the account and presents a sign-in path. Activation does not create a session.
4. Correct credentials for an inactive account do not sign in. Incorrect credentials retain generic feedback.
5. An expired, used, or replaced link cannot activate an account. The visitor can request a new link.
6. Resend accepts a valid email and gives neutral feedback regardless of account state or existence. Only inactive accounts receive mail, at most three sends per address per hour.
7. Existing accounts retain access, including accounts whose `email_verified_at` was previously empty.

## Requirements

- Use `email_verified_at` as the activation state. New registrations leave it empty.
- Use single-use 24-hour links. Issuing a new link replaces the previous one.
- Protect every authenticated API route against inactive accounts, including sessions established outside the normal login flow.
- Queue a responsive branded HTML email with a plain-text version. Localize PT-BR by default and English when registration requests it. Never log activation tokens.
- Keep registration validation, email normalization, password rules, CSRF handling, and login throttling consistent with the existing authentication feature.
- Provide accessible confirmation, activation result, resend, and inactive sign-in guidance in both supported UI languages.

## Acceptance

- Register -> check email -> activate -> sign in reaches the dashboard.
- Before activation, login and protected data access fail without a session.
- Expired, replaced, malformed, and reused links cannot activate an account.
- Resend responses do not reveal whether an email belongs to an account.
- Backfilled existing accounts still sign in after deployment.
