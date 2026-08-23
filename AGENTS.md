<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

## Zunera Workspace Layout

- Specs: `zunera-specs/`
- Backend: `../zunera-backend/`
- Frontend: `../zunera-frontend/`
- For every feature: specify, plan, and implement backend first; frontend follows
  only after backend API contract, authorization, validation, and tests are done.
- Feature branches use identical names in specs, backend, and frontend. Before
  editing either application, verify its local branch matches specification branch.
