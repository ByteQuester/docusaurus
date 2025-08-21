---
id: containers-and-helm
slug: /tasks/20-containers-and-policies/containers-and-helm
sidebar_position: 1
---

# Containers and Helm

### CT-001 — Containerize frontend

- **Dependencies:** REPO-002
- **Deliverables:** multi-stage Dockerfile; `.dockerignore`; local run instructions.
- **Acceptance:** image serves app locally.

### CT-002 — Containerize sample backend

- **Dependencies:** BE-001
- **Deliverables:** Dockerfile; `.dockerignore`; health endpoint.
- **Acceptance:** container responds 200 on `/health`.

### HELM-001 — Frontend Helm chart

- **Dependencies:** CT-001
- **Deliverables:** `charts/frontend` with Deployment, Service, Ingress, HPA; `values.yaml`, env overlays.
- **Acceptance:** `helm lint` passes; `helm template` renders.

### HELM-002 — Backend Helm chart

- **Dependencies:** CT-002
- **Deliverables:** `charts/backend-sample` with Service, Ingress path `/api`, ConfigMap for settings.
- **Acceptance:** `helm lint` passes; `helm template` renders.

### HELM-003 — Publish Helm repo

- **Dependencies:** HELM-001, HELM-002, CI-003
- **Deliverables:** package charts, generate index, publish to static hosting (e.g., GitHub Pages or bucket).
- **Acceptance:** `helm repo add` and `helm search repo` work.

### HELM-004 — Chart testing automation

- **Dependencies:** CI-003
- **Deliverables:** chart-testing tooling in CI; basic lint/test pipeline.
- **Acceptance:** PRs trigger chart tests and gate merges.

### HELM-005 — Values management strategy

- **Dependencies:** HELM-001
- **Deliverables:** `values-{dev,prod}.yaml`; doc on config per env; sanitize sensitive values.
- **Acceptance:** renders for both envs; no secrets committed.
