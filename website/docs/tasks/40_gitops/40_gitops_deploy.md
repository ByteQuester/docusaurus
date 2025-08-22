---
id: 40-gitops-deploy
slug: /tasks/40-gitops/deploy
sidebar_position: 1
---

# Gitops Deploy

### GITOPS-001 — Select GitOps controller

- **Dependencies**: `INF-002`
- **Deliverables**:
  - Choose between Argo CD or Flux.
  - Create a decision document outlining the rationale.
- **Acceptance Criteria**:
  - The chosen tool and rationale are formally documented.

### GITOPS-002 — Install controller

- **Dependencies**: `GITOPS-001`
- **Deliverables**:
  - Install Argo CD in the `argocd` namespace.
  - Secure administrative access.
- **Acceptance Criteria**:
  - The controller UI is reachable with restricted authentication.
  - All associated pods are healthy.

### GITOPS-003 — Define Applications

- **Dependencies**: `GITOPS-002`, `HELM-001`
- **Deliverables**:
  - Define frontend and backend applications in `infra/gitops/apps/*`.
  - Specify application sources and values.
- **Acceptance Criteria**:
  - Applications are reported as "Synced" and "Healthy" in the controller UI.

### GITOPS-004 — Sync and health policies

- **Dependencies**: `GITOPS-003`
- **Deliverables**:
  - Configure automated synchronization and health checks.
  - Enable self-healing capabilities.
- **Acceptance Criteria**:
  - Any configuration drift is automatically corrected in a test environment.

### GITOPS-005 — App-of-Apps pattern (optional)

- **Dependencies**: `GITOPS-003`
- **Deliverables**:
  - Implement a parent Application to aggregate all app definitions.
- **Acceptance Criteria**:
  - Child applications are successfully managed from the root application.

### GITOPS-006 — Promotion flow dev→prod

- **Dependencies**: `GITOPS-003`, `HELM-005`
- **Deliverables**:
  - Document the PR-based process for promoting values between environments.
- **Acceptance Criteria**:
  - A test promotion successfully changes an image tag in the production environment via a pull request.

### GITOPS-007 — Correct Argo CD Application destination

- **Dependencies**: `GITOPS-003`
- **Deliverables**:
  - Correct the `spec.destination.server` in `infra/gitops/apps/frontend.yaml` to `https://kubernetes.default.svc`.
- **Acceptance Criteria**:
  - Argo CD accepts the Application configuration and displays a valid destination cluster.
