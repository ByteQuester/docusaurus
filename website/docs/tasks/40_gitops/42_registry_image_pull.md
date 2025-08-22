---
id: 42-registry-image-pull
slug: /tasks/40-gitops/registry-image-pull
sidebar_position: 3
---

# Registry Image Pull Recovery

### REG-001 — Identify correct image repo and tags

- **Dependencies**: `CI-002`
- **Deliverables**:
  - Determine the correct image repository and tag for the chart being deployed.
- **Acceptance Criteria**:
  - The exact repository and tag are known and exist.

### REG-002 — Ensure image exists

- **Dependencies**: `CI-002`
- **Deliverables**:
  - Trigger a new CI build to push a fresh image if one does not exist.
- **Acceptance Criteria**:
  - The target tag exists in the GitHub Container Registry (GHCR).

### REG-003 — Create/refresh registry pull secrets

- **Dependencies**: None
- **Deliverables**:
  - Create a `docker-registry` secret in the `frontend-dev` namespace with a PAT that has `read:packages` scope.
- **Acceptance Criteria**:
  - The `ghcr-credentials` secret exists in the `frontend-dev` namespace.

### REG-004 — Wire imagePullSecrets to workloads

- **Dependencies**: `REG-003`
- **Deliverables**:
  - Wire the `imagePullSecrets` to the workloads using either Helm values or by patching the default ServiceAccount.
- **Acceptance Criteria**:
  - The `imagePullSecrets` are shown in the default ServiceAccount or the rendered Deployment includes the pull secret.

### REG-005 — Update Helm values with correct image

- **Dependencies**: `REG-001`
- **Deliverables**:
  - Update the Helm values with the correct image repository and tag.
- **Acceptance Criteria**:
  - `helm template` renders the expected image reference.

### REG-006 — Redeploy and verify

- **Dependencies**: `REG-004`, `REG-005`
- **Deliverables**:
  - Redeploy the application and verify that the pods are running correctly.
- **Acceptance Criteria**:
  - The frontend pods become Ready and there is no `ImagePullBackOff` error.

### REG-007 — Document registry auth and image mapping

- **Dependencies**: `REG-006`
- **Deliverables**:
  - Create `docs/docs/registry-auth.md` to document the registry authentication and image mapping process.
- **Acceptance Criteria**:
  - The document is created and referenced from other relevant documents.

### REG-008 — Optional: make images public or grant access

- **Dependencies**: None
- **Deliverables**:
  - Consider making the GHCR package public or ensuring the PAT has the correct permissions.
- **Acceptance Criteria**:
  - Image pulls succeed with the appropriate level of access.
