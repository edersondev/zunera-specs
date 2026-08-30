# Research: User Authentication

## Decisions

### First-party session authentication
- **Decision**: Use Laravel Sanctum's stateful, cookie-backed session flow for the browser SPA. The browser obtains a CSRF cookie, then calls versioned JSON auth routes; no bearer token is stored in the browser.
- **Rationale**: It matches the existing Laravel session guard and Sanctum setup, limits credential exposure, and supports server-enforced idle and absolute expiry.
- **Alternatives considered**: Personal access tokens were rejected because they add long-lived client secrets and do not improve this first-party browser flow.

### Password policy and compromise checks
- **Decision**: Enforce 15–64 characters, printable characters including spaces, confirmation, and Laravel's uncompromised-password validator. Keep the verifier behind a service boundary so tests can mock it and failures are observable.
- **Rationale**: NIST SP 800-63B recommends at least 15 characters for single-factor passwords and long-password support; the spec requires rejecting common or compromised choices.
- **Alternatives considered**: Arbitrary upper/lower/number composition was rejected because it harms usability without satisfying the stated requirement. A new password-scanning package was rejected to avoid an unapproved dependency.

### Email-only registration
- **Decision**: Make the existing `users.name` column nullable in a backend migration; registration accepts only email, password, and confirmation. Do not synthesize a display name from the email address.
- **Rationale**: The user need and LGPD minimization explicitly require only authentication data. The starter schema's required name is an implementation mismatch, not a user requirement.
- **Alternatives considered**: Asking for a name expands the flow; deriving one from email leaks or guesses personal data.

### Recovery privacy and delivery
- **Decision**: Return one neutral recovery response for every well-formed request, queue the notification after commit, cap sends at three per normalized address per hour, and use the framework's 60-minute, newest-token-only broker behavior. Reset sends a separate security notification and invalidates all sessions.
- **Rationale**: This prevents account enumeration, keeps the request responsive, and directly implements the clarified security requirements.
- **Alternatives considered**: Synchronous mail and account-specific responses were rejected because they expose timing/account state and make the request dependent on SMTP latency.

### Session policy
- **Decision**: Configure 15-minute idle expiry and enforce an 8-hour absolute expiry in backend middleware using signed-in session metadata. Return both authoritative deadlines; the frontend warns one minute before idle expiry and can continue only through an authenticated backend call.
- **Rationale**: Laravel's normal session lifetime is idle-only; a small middleware closes the absolute-time gap while keeping the server authoritative.
- **Alternatives considered**: Client-only timers were rejected because users can tamper with them and they cannot invalidate an expired server session.

### Progressive sign-in throttling
- **Decision**: Use a cache-backed limiter keyed by a non-reversible normalized-email hash with IP as secondary context. After five failures in 15 minutes, apply 1/2/4/8/15-minute delays (cap 15), return generic errors plus `Retry-After`, and clear on success.
- **Rationale**: This meets the clarified rule while limiting denial-of-service risk from locking accounts by email alone.
- **Alternatives considered**: Permanent lockout was rejected because OWASP warns it can be abused to deny service and reveal account state.

## References

- Laravel 13 documentation: Sanctum SPA authentication, session regeneration/logout, validation, rate limiting, password reset notifications and queued notifications: https://laravel.com/docs/13.x
- NIST SP 800-63B, memorized secrets: https://pages.nist.gov/800-63-4/sp800-63b.html
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Session Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Forgot Password Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html
- Brazil ANPD, LGPD guidance: https://www.gov.br/anpd/pt-br
