---
id: gitops-and-deployment
slug: /tasks/40-gitops/gitops-and-deployment
sidebar_position: 1
---

# GitOps and Deployment

### GITOPS-001 — Select GitOps controller

- **Dependencies:** INF-002
- **Deliverables:** choose Argo CD or Flux; decision doc.
- **Acceptance:** documented rationale and chosen tool.

### GITOPS-002 — Install controller

- **Dependencies:** GITOPS-001
- **Deliverables:** install Argo CD in `argocd` namespace; admin access secured.
- **Acceptance:** controller UI reachable (auth restricted); pods healthy.

### GITOPS-003 — Define Applications

- **Dependencies:** GITOPS-002, HELM-001
- **Deliverables:** `infra/gitops/apps/*` defining frontend and backend apps with sources and values.
- **Acceptance:** apps show Synced/Healthy.

### GITOPS-004 — Sync and health policies

- **Dependencies:** GITOPS-003
- **Deliverables:** automated sync and health checks; self-heal enabled.
- **Acceptance:** drift corrected automatically on test.

### GITOPS-005 — App-of-Apps pattern (optional)

- **Dependencies:** GITOPS-003
- **Deliverables:** parent Application aggregating app definitions.
- **Acceptance:** children managed from the root app.

### GITOPS-006 — Promotion flow dev→prod

- **Dependencies:** GITOPS-003, HELM-005
- **Deliverables:** document PR-based value promotion between envs.
- **Acceptance:** a test promotion changes image tag in prod via PR.

### GITOPS-007 — Correct Argo CD Application destination

- **Dependencies:** GITOPS-003
- **Deliverables:** fix `infra/gitops/apps/frontend.yaml` `spec.destination.server` to `https://kubernetes.default.svc`.
- **Acceptance:** Argo CD accepts the Application and shows a valid destination cluster.
