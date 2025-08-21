# Backend Onboarding

## BE-001 — Sample backend service scaffold

- Dependencies: REPO-001
- Deliverables: `apps/backend-sample` with health and one demo route; README for local dev.
- Acceptance: `npm run dev` or equivalent serves endpoints locally.

## BE-002 — Dockerize backend

- Dependencies: BE-001
- Deliverables: Dockerfile and `.dockerignore` for backend.
- Acceptance: container responds 200 on `/health`.

## BE-003 — Backend Helm chart

- Dependencies: BE-002
- Deliverables: `charts/backend-sample` with Service, Ingress at `/api`, and ConfigMap.
- Acceptance: `helm lint` passes; `helm template` renders.

## BE-004 — Ingress integration and CORS

- Dependencies: BE-003, HELM-001
- Deliverables: path-based routing under frontend domain; CORS settings implemented as needed.
- Acceptance: `https://<domain>/api/health` returns 200; frontend can call API.

## BE-005 — OpenAPI and versioning [optional]

- Dependencies: BE-001
- Deliverables: OpenAPI spec; docs page; version prefix in routes.
- Acceptance: spec published; linted.

## BE-006 — Config and flags

- Dependencies: BE-003
- Deliverables: environment variables documented; default values via ConfigMap; secrets via ESO.
- Acceptance: settings override via Helm values.

## BE-007 — Tests and quality gates

- Dependencies: BE-001
- Deliverables: basic unit/integration tests; CI job to run them.
- Acceptance: CI fails on failing tests; coverage reported (optional).
