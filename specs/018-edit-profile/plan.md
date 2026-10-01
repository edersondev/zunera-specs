# Implementation Plan: Edit Profile

**Branch**: `018-edit-profile` | **Date**: 2026-10-01 | **Spec**: [spec.md](spec.md)  
**Input**: Edit profile and password change, with disabled email.

**Branch Coordination**: `018-edit-profile` is active in specs, backend, and frontend.

## Summary

Add a protected profile-name mutation and a separate signed-in password-change mutation. Expose them through separate Edit profile and Change password dialogs in the account menu. Email is display-only.

## Technical Context

**Language/Version**: PHP 8.3 / Laravel 13.26; JavaScript / Vue 3  
**Primary Dependencies**: Existing Sanctum, Eloquent, Axios, Pinia, Element Plus  
**Storage**: Existing users, database sessions, password-reset tokens  
**Testing**: PHPUnit feature tests; Vitest; Playwright; Pint; lint/build  
**Target Platform**: Authenticated responsive web app, Portuguese and English  
**Project Type**: Full-stack feature  
**Performance Goals**: Dialog save feedback within normal API response time  
**Constraints**: No new packages, secrets, tables, or email mutation  
**Scale/Scope**: Current user profile and password only

## Delivery Scope and Order

1. **Backend** — Protected API contract, Form Requests, service, session/token cleanup, notification, and tests.
2. **Frontend** — Two account-menu entries and focused dialogs, auth transport/store updates, localization, and tests after backend contract and tests pass.

## Constitution Check

- [x] Protected routes, Form Requests, and UserResource define API boundary.
- [x] Backend contract, authorization, validation, and tests precede frontend.
- [x] All three repositories use matching feature branches.
- [x] Authentication service owns mutations; no repository needed for simple indexed user/session operations.
- [x] Vue Composition API, existing JavaScript convention, Axios service, shared Pinia session state, and Element Plus dialog/forms.
- [x] Feature/contract tests, frontend tests, and isolated browser journey cover changes.
- [x] No upload or new secret; passwords never enter responses/logs.

## Design

- Backend: `PATCH /api/v1/auth/profile` accepts name alone and rejects other keys. `PATCH /api/v1/auth/password` validates current password and confirmed safe new password, limits failed checks to five per 15 minutes, rotates hash, invalidates reset tokens, retains current session, deletes other sessions, and queues existing notice. No migration.
- Frontend: `AppHeader` emits Edit profile or Change password; `AppShell` mounts both dialogs. Profile dialog preloads current name/email and disables email. Password dialog handles only the password save. `authService` uses CSRF-aware API requests; `sessionStore` updates shared user after name save. Closing clears errors and passwords and restores focus.
- Test name changes, unsupported email, unauthenticated/expired access, validation, current-password failures and limits, sessions, tokens, mail, localization, keyboard, and 320px/200% layout. Follow [quickstart.md](quickstart.md).

## Project Structure

`specs/018-edit-profile/` holds spec, plan, contract, research, data model, quickstart, checklist, and tasks. Backend extends existing Authentication service/requests/controller/tests. Frontend extends header, app shell, auth service/store, and adds focused profile and password dialogs.
