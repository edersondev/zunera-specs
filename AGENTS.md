<!-- SPECKIT START -->
For additional context about technologies, project structure, shell commands,
and implementation order, read [specs/010-credit-cards/plan.md](specs/010-credit-cards/plan.md).
<!-- SPECKIT END -->

## Zunera Workspace Layout

- Specs: `zunera-specs/`
- Backend: `../zunera-backend/`
- Frontend: `../zunera-frontend/`
- For every feature: specify, plan, and implement backend first; frontend follows
  only after backend API contract, authorization, validation, and tests are done.
- Feature branches use identical names in specs, backend, and frontend. Before
  editing either application, verify its local branch matches specification branch.
- When `speckit-specify` creates a spec branch, create matching branches only in
  applications affected by that spec: backend-only changes get a backend branch
  only; frontend-only changes get a frontend branch only; full-stack changes get
  both.
- For every frontend implementation planned with `speckit-plan`, follow these
  design documents:
  - `docs/design/design-foundation.md`
  - `docs/design/app-shell.md`
  - `docs/design/navigation.md`
  - `docs/design/components.md`
