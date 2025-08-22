---
id: 70-backend-onboarding
slug: /tasks/70-onboarding/backend
sidebar_position: 1
---

# Backend Onboarding

### TASK-001 — Sample backend service scaffold

- **Dependencies**: `REPO-001`
- **Deliverables**: `apps/backend-sample` with health and one demo route; README for local dev.
- **Acceptance Criteria**: `npm run dev` or equivalent serves endpoints locally.

### TASK-002 — Dockerize backend

- **Dependencies**: `BE-001`
- **Deliverables**: Dockerfile and `.dockerignore` for backend.
- **Acceptance Criteria**: container responds 200 on `/health`.

### TASK-003 — Backend Helm chart

- **Dependencies**: `BE-002`
- **Deliverables**: `charts/backend-sample` with Service, Ingress at `/api`, and ConfigMap.
- **Acceptance Criteria**: `helm lint` passes; `helm template` renders.

### TASK-004 — Ingress integration and CORS

- **Dependencies**: `BE-003`, `HELM-001`
- **Deliverables**: path-based routing under frontend domain; CORS settings implemented as needed.
- **Acceptance Criteria**: `https://<domain>/api/health` returns 200; frontend can call API.

### TASK-005 — OpenAPI and versioning [optional]

- **Dependencies**: `BE-001`
- **Deliverables**: OpenAPI spec; docs page; version prefix in routes.
- **Acceptance Criteria**: spec published; linted.

### TASK-006 — Config and flags

- **Dependencies**: `BE-003`
- **Deliverables**: environment variables documented; default values via ConfigMap; secrets via ESO.
- **Acceptance Criteria**: settings override via Helm values.

### TASK-007 — Tests and quality gates

- **Dependencies**: `BE-001`
- **Deliverables**: basic unit/integration tests; CI job to run them.
- **Acceptance Criteria**: CI fails on failing tests; coverage reported (optional).
