# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Branch Coordination**: Create and switch to this exact branch name in
`zunera-specs`, `../zunera-backend`, and `../zunera-frontend` before work starts.

**Note**: This template is filled in by the `/speckit-plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]  
**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]  
**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]  
**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]  
**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]
**Project Type**: [e.g., library/cli/web-service/mobile-app/compiler/desktop-app or NEEDS CLARIFICATION]  
**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]  
**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]  
**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]

## Delivery Scope and Order

Document both scopes in this exact order:

1. **Backend** — Laravel API contracts, authorization, validation, services,
   persistence, and backend tests in `../zunera-backend`.
2. **Frontend** — Vue views, services, Pinia state, Element Plus UI, and frontend
   tests in `../zunera-frontend`.

Frontend implementation MUST start only after the relevant backend API contract,
authorization, validation, and tests are complete.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- [ ] API boundaries identify Form Requests, middleware, API Resources, and any
      upload validation/sanitization needed.
- [ ] Backend scope and API contract are complete before frontend work; frontend
      scope consumes that contract from `../zunera-frontend`.
- [ ] Feature branch name matches in specs, backend, and frontend repositories.
- [ ] Business rules have service ownership; DTO and repository use is justified.
- [ ] Frontend design uses Composition API, services for API access, Pinia only
      for shared state, and Element Plus for standard interface controls.
- [ ] Test and API-contract coverage covers every changed behavior; critical
      UI-to-API journeys include isolated Playwright end-to-end coverage.
- [ ] Security, environment configuration, and scope constraints are recorded.
- [ ] Any exception to the constitution is listed in Complexity Tracking.

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (sibling repositories)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
../zunera-backend/
├── app/
│   ├── Http/
│   └── Services/
└── tests/

../zunera-frontend/
├── src/
│   ├── components/
│   ├── services/
│   └── stores/
└── e2e/
```

**Structure Decision**: Backend work in `../zunera-backend` precedes frontend
work in `../zunera-frontend`; document exact feature paths under each.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
