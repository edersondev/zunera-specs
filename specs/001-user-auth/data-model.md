# Data Model: User Authentication

## User

- `id`: existing primary key.
- `name`: nullable legacy display field; not collected or serialized by this feature.
- `email`: unique normalized (trimmed, lowercase) login identifier.
- `password`: framework-hashed secret; never serialized or logged.
- `email_verified_at`: remains nullable; this feature does not add email verification.
- timestamps and remember token: existing framework fields.

## Authenticated session

- Existing database session row keyed by session ID and user ID.
- Session metadata stores `authenticated_at` and `absolute_expires_at` (server timestamps).
- Idle expiry is 15 minutes from last activity; absolute expiry is 8 hours from authentication and never extends.
- API exposes only `idle_expires_at` and `absolute_expires_at`; cookies and session identifiers remain HttpOnly.

## Password recovery token

- Existing broker table keyed by normalized email.
- Token is single-use, valid for 60 minutes, and replaced when a newer request is created.
- Raw token appears only in the emailed link and reset form route state; it is never persisted client-side or returned by API responses.

## Rate-limit records

- Cache entries, not user records, track failed sign-ins and recovery sends.
- Keys use a one-way hash of normalized email plus secondary IP context; values include attempt count and expiry.
- No password, token, or recoverable email content is stored in these keys.

## Authentication mail delivery event

- `event_id`: unique opaque event identifier used for idempotency.
- `message_id`: lowercase RFC 4122 UUID added to outgoing auth mail as
  `X-Zunera-Message-ID` and echoed unchanged by the canonical event.
- `status`: `delivered`, `bounced`, `deferred`, or `rejected`.
- `occurred_at` and `received_at`: provider/gateway and application timestamps.
- The signed canonical event contains no recipient address, subject, body, token,
  or provider-specific payload and is retained only for the documented metric window.

## Relationships and lifecycle

- A user has many authenticated sessions and at most one active broker recovery token.
- Registration creates a user and authenticates one session transactionally.
- Sign-out invalidates only the current session.
- Successful password reset changes the hash, consumes the token, deletes all sessions for the user, and queues a security notification after commit.
- Recovery notification attempts correlate to zero or more idempotent delivery
  events through the opaque `message_id`.
