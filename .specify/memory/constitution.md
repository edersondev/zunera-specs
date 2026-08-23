<!--
Sync Impact Report
- Version change: 1.3.0 → 1.4.0
- Modified principles: Delivery Workflow and Quality Gates → Delivery Workflow
  and Quality Gates (cross-repository feature branch rule added)
- Added sections: none
- Removed sections: none
- Templates requiring updates:
  - ✅ .specify/templates/plan-template.md
  - ✅ .specify/templates/spec-template.md
  - ✅ .specify/templates/tasks-template.md
  - ✅ .specify/extensions/git/scripts/bash/create-new-feature.sh
  - ✅ .specify/extensions/git/scripts/powershell/create-new-feature.ps1
  - ✅ .specify/extensions/git/commands/speckit.git.feature.md
  - ✅ .specify/extensions/git/README.md
  - ✅ .specify/templates/commands/ (directory absent; no command templates to update)
  - ✅ AGENTS.md
- Follow-up TODOs:
  - TODO(RATIFICATION_DATE): Original constitution adoption date is not recorded.
-->
# Zunera Constitution

## Core Principles

### I. Secure, Explicit API Boundaries
All backend inputs MUST be validated through Laravel Form Requests or equivalent
validation. Protected routes MUST use appropriate middleware, responses MUST use
API Resources, and upload handling MUST sanitize and validate files. Controllers
MUST remain thin; security-sensitive behavior belongs in tested services. This
keeps authorization, validation, and public contracts consistent and reviewable.

### II. Service-Layer Domain Logic
Business logic MUST live in `App\\Services` or an equally scoped domain service.
Use DTOs when inputs contain more than one meaningful field, exceptions rather
than return codes for exceptional paths, and repositories only for genuinely
complex data access. Services MUST NOT create global state. This preserves clear
domain ownership and testability without needless abstraction.

### III. Vue 3 Composition and State Discipline
Vue interfaces MUST use Vue 3 Composition API with `<script setup>`. Components
MUST keep API calls in services and UI state shared across views in Pinia stores;
components MUST NOT contain domain or transport logic. Use Axios for API access
and Tailwind CSS for styling unless an existing project convention requires a
compatible alternative. This keeps UI code composable, predictable, and easy to
test.

### IV. Testable, Contract-Safe Changes
Every behavior change MUST include proportionate automated coverage: Laravel
feature and unit tests for backend behavior, and frontend service or composable
tests for frontend behavior. Changes to HTTP endpoints, request validation, or
response shapes MUST include contract or feature coverage. External dependencies
MUST be mocked where practical. Tests provide executable evidence that a change
preserves user-facing and integration contracts. Critical user journeys that
cross the interface and API MUST have Playwright end-to-end coverage. Playwright
tests MUST use accessible selectors and preserve scenario isolation.

### V. Focused, Maintainable Delivery
Features MUST respect existing structure, avoid unrelated refactors, and avoid
new packages unless explicitly approved. Plans and implementations MUST state
scope boundaries, preserve clear naming, and justify material complexity in the
Constitution Check. This limits accidental coupling and keeps delivery work
reviewable.

## Platform Constraints

Backend work MUST follow PSR-12 and Laravel conventions. Frontend work MUST use
Vue 3, Pinia, Axios, Tailwind CSS, and Element Plus as project defaults. Element
Plus components MUST be preferred for standard interface controls and MUST follow
the project's configured theme and accessibility conventions. Secrets MUST be
read from environment variables and MUST NOT enter source control.
Implementations MUST follow OWASP fundamentals, including authorization, input
validation, and safe error handling.

## Delivery Workflow and Quality Gates

Each feature specification MUST define independently testable user scenarios,
functional requirements, edge cases, security implications, and measurable
outcomes. Every specification, plan, and task set MUST include explicit Backend
and Frontend scopes in that order. Backend work MUST define and implement its API
contract, authorization, validation, services, and tests before frontend work
starts; frontend work MUST consume the established backend contract. Each
feature branch created for a new specification MUST use the identical name in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend`; work MUST start
on the corresponding backend and frontend branch. Each
implementation plan MUST pass the Constitution Check before research and again
after design. Tasks MUST include required verification work next to relevant
implementation work, then run relevant test and style commands before
completion. Reviewers MUST verify API boundaries, service ownership, Vue state
boundaries, security, scope, delivery order, and test evidence.

## Governance

This constitution supersedes conflicting project practices. Amendments MUST be
documented in this file, include a Sync Impact Report, and update affected
templates and runtime guidance in the same change. Versioning uses semantic
versioning: MAJOR for incompatible governance changes, MINOR for principles or
material guidance added, and PATCH for clarifications. Every plan, task set, and
review MUST assess compliance; exceptions MUST be explicit and justified in the
plan's Complexity Tracking section. Runtime guidance lives in `AGENTS.md` and
the active feature `plan.md`.

**Version**: 1.4.0 | **Ratified**: TODO(RATIFICATION_DATE): Original adoption date unknown | **Last Amended**: 2026-08-23
