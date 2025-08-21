---
id: repo-and-workspace
slug: /tasks/00-setup/repo-and-workspace
sidebar_position: 2
---

# Repository and Workspace

:::info Ordered tasks to establish a maintainable monorepo foundation. :::

### REPO-001 — Bootstrap repository skeleton

- **Dependencies:** none
- **Deliverables:** directories (`apps/`, `charts/`, `infra/`, `infra/gitops/`, `docs/`, `.github/workflows/`, `ops/`, `packages/`), root `.gitignore`, `.dockerignore`, `LICENSE`, `README.md`.
- **Acceptance:** tree matches layout; no app code changes.

### REPO-002 — Integrate existing frontend into `apps/frontend`

- **Dependencies:** REPO-001
- **Deliverables:** copy scaffolded frontend to `apps/frontend` with its configs; `apps/frontend/README.md` for local dev.
- **Acceptance:** local dev command serves the app.

### REPO-003 — Configure workspaces

- **Dependencies:** REPO-001
- **Deliverables:** `pnpm-workspace.yaml` including `apps/*`, `packages/*`; Node version file; root README section.
- **Acceptance:** `pnpm -w install` detects workspaces successfully.

### REPO-004 — Linting and formatting baseline

- **Dependencies:** REPO-003
- **Deliverables:** root Prettier config, `.editorconfig`, shared ESLint package in `packages/eslint-config`, scripts.
- **Acceptance:** `pnpm -w lint` and `pnpm -w format` run cleanly.

### REPO-005 — Pre-commit hooks

- **Dependencies:** REPO-004
- **Deliverables:** pre-commit config for lint/format; hook installation instructions.
- **Acceptance:** committing triggers hooks; rejects unformatted code.

### REPO-006 — Makefile task runners

- **Dependencies:** REPO-003
- **Deliverables:** `Makefile` with common targets (install, lint, build, test, image, helm:lint, helm:template).
- **Acceptance:** `make help` lists targets; targets execute successfully.

### REPO-007 — Governance

- **Dependencies:** REPO-001
- **Deliverables:** `CODEOWNERS`, PR/issue templates, `CONTRIBUTING.md`, `SECURITY.md`.
- **Acceptance:** templates appear on new PRs; policies documented.
