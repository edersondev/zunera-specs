<!-- SPECKIT START -->
For additional context about technologies, project structure, shell commands,
and implementation order, read [specs/013-transactions-activity-redesign/plan.md](specs/013-transactions-activity-redesign/plan.md).
<!-- SPECKIT END -->

## Zunera Workspace Layout

- Specs: `zunera-specs/`
- Backend: `../zunera-backend/`
- Frontend: `../zunera-frontend/`

## Test Commands

- **Backend (preferred)**: the development stack already runs in Docker, so run
  the suite inside the live container instead of on the host:

  ```bash
  docker exec zunera-backend-app-1 php artisan test
  docker exec zunera-backend-app-1 php artisan test --filter=CreditCards
  docker exec zunera-backend-app-1 vendor/bin/pint --format=agent
  ```

  The project root is mounted into that container, so host edits are picked up
  immediately and no rebuild is needed. Start or stop the stack from
  `../zunera-backend/` with `./start.sh` and `./stop.sh` when the container is
  not running. In a sandboxed Codex session, Docker commands must run with
  escalated permissions because the Docker socket is not reachable otherwise.
- **Frontend**: `npm run test:unit -- --run` (Vitest), `CI=1 npm run test:e2e --
  e2e/<feature>.spec.js` (Playwright; build first because CI mode uses the
  preview server), and `npm run lint` from `../zunera-frontend/`. Playwright
  browsers are already installed locally.
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
