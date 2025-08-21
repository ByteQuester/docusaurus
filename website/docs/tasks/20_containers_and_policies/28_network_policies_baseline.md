---
id: network-policies-baseline
slug: /tasks/20-containers-and-policies/network-policies-baseline
sidebar_position: 2
---

# Network Policies Baseline

:::info **Goal:** Enforce least-privilege traffic in EKS while keeping the sample frontend/backend functional and observable. :::

## Subtask Index

- [NPOL-000 — Verify NetworkPolicy enforcement engine](./network-policies/NPOL-000_verify-enforcement-engine.md)
- [NPOL-001 — Namespace and label conventions](./network-policies/NPOL-001_namespace-and-labels.md)
- [NPOL-002 — Default deny (ingress and egress)](./network-policies/NPOL-002_default-deny.md)
- [NPOL-003 — Allow DNS egress](./network-policies/NPOL-003_allow-dns.md)
- [NPOL-004 — Allow frontend → backend](./network-policies/NPOL-004_frontend-to-backend.md)
- [NPOL-005 — Allow ingress-controller → backend](./network-policies/NPOL-005_ingress-to-backend.md)
- [NPOL-006 — Allow monitoring scrapes](./network-policies/NPOL-006_monitoring-scrapes.md)
- [NPOL-007 — Controlled egress to Internet](./network-policies/NPOL-007_controlled-egress.md)
- [NPOL-008 — Apply order, tests, rollback](./network-policies/NPOL-008_apply-validate-rollback.md)
- [NPOL-TRIAGE-001 — Confirm enforcement & Kyverno](./network-policies/NPOL-TRIAGE-001_enforcement-and-kyverno.md)
- [NPOL-TRIAGE-002 — Verify default-deny & DNS](./network-policies/NPOL-TRIAGE-002_verify-deny-and-dns.md)
- [NPOL-TRIAGE-003 — Verify allow rules](./network-policies/NPOL-TRIAGE-003_verify-allows.md)
- [APP-NET-001 — Backend Service/Deployment alignment](./network-policies/APP-NET-001_backend-service-alignment.md)
- [APP-NET-002 — App bind address & image/tag propagation](./network-policies/APP-NET-002_bind-and-image.md)
- [GITOPS-ALIGN-001 — Argo application sanity](./network-policies/GITOPS-ALIGN-001_argo-sanity.md)
- [NPOL-VALIDATE-001 — Functional checks under policies](./network-policies/NPOL-VALIDATE-001_functional-checks.md)
- [KYVERNO-REVERT-001 — Tighten Kyverno baseline post-install](./network-policies/KYVERNO-REVERT-001_tighten-kyverno.md)
- [OBS-DOCS-001 — Record selectors and enforcement engine](./network-policies/OBS-DOCS-001_selectors-and-engine.md)
- [ROLLBACK-READY-001 — Single-command rollback plan](./network-policies/ROLLBACK-READY-001_single-command.md)

---

## Policy Implementation Tasks

### NPOL-000 — Verify NetworkPolicy enforcement engine

- **Dependencies:** AWSZ-006 (kubectl context)
- **Steps:**
  - Confirm a policy engine is installed (Calico or Cilium). The AWS VPC CNI alone does not enforce NetworkPolicy.
  - If missing, install Calico (quick path) or Cilium (optional) via Helm/operator.
  - Record the choice and version in `../observability.md`.
- **Acceptance:** `kubectl get pods -n kube-system` shows Calico or Cilium components Ready; a simple default-deny test is enforceable.

### NPOL-001 — Namespace and label conventions

- **Dependencies:** AWSZ-015, AWSZ-016
- **Steps:**
  - Ensure namespaces exist and are consistently labeled: `frontend-dev`, `backend-dev`, `ingress` (ingress-nginx), `monitoring`.
  - Ensure Deployments have stable labels used by policies (e.g., `app=frontend`, `app=backend`).
  - Document chosen selectors in `../observability.md`.
- **Acceptance:** Namespaces and app labels exist and match intended selectors.

### NPOL-002 — Default deny (ingress and egress) in app namespaces

- **Dependencies:** NPOL-001
- **Steps:**
  - Create `infra/manifests/network-policies/frontend-dev/00-deny-all.yaml` to deny ingress and egress for all pods in `frontend-dev`.
  - Create `infra/manifests/network-policies/backend-dev/00-deny-all.yaml` to deny ingress and egress for all pods in `backend-dev`.
- **Acceptance:** Applying only these policies blocks inbound/outbound traffic for app pods (health checks fail as expected).

### NPOL-003 — Allow DNS egress (CoreDNS)

- **Dependencies:** NPOL-002
- **Steps:**
  - Add `allow-dns.yaml` in each app namespace directory to allow egress to `kube-system` CoreDNS on TCP/UDP 53.
  - Use a NamespaceSelector for `kube-system` and PodSelector for `k8s-app=kube-dns` (or the chart’s labels).
- **Acceptance:** `nslookup` from app pods resolves; no other egress is allowed yet.

### NPOL-004 — Allow frontend → backend service traffic

- **Dependencies:** NPOL-002
- **Steps:**
  - In `backend-dev`, create `10-allow-from-frontend.yaml` allowing ingress from namespace `frontend-dev` to pods labeled `app=backend` on required ports (e.g., 80/443 or app port).
  - Optionally scope to ServiceAccount or additional pod labels for least privilege.
- **Acceptance:** Frontend can call backend; `/api/health` succeeds when policies applied with this allow.

### NPOL-005 — Allow ingress-controller → backend traffic

- **Dependencies:** NPOL-002
- **Steps:**
  - In `backend-dev`, add `15-allow-from-ingress.yaml` permitting ingress from `ingress` namespace (ingress-nginx controller pods) to backend service ports.
  - Match ingress controller labels (e.g., `app.kubernetes.io/name=ingress-nginx`).
- **Acceptance:** External requests through the Ingress reach backend; site remains functional under default-deny.

### NPOL-006 — Allow monitoring scrapes (optional but recommended)

- **Dependencies:** NPOL-002, AWSZ-024
- **Steps:**
  - Create `20-allow-from-monitoring.yaml` in app namespaces to allow ingress from `monitoring` namespace to pods’ metrics ports (e.g., 8080/`/metrics`), limited to pods exposing metrics.
  - Align selectors with your ServiceMonitor/PodMonitor configuration.
- **Acceptance:** Prometheus targets for app metrics show `Up`; Grafana dashboards populate.

### NPOL-007 — Controlled egress to Internet (only where needed)

- **Dependencies:** NPOL-002
- **Steps:**
  - For pods that must reach external services, add `30-allow-egress-external.yaml` to permit egress to required CIDRs/ports only.
  - Keep frontend with minimal egress unless necessary (e.g., external APIs, package registries).
- **Acceptance:** Required external calls succeed; unnecessary egress remains blocked.

### NPOL-008 — Apply order, smoke tests, and rollback

- **Dependencies:** NPOL-003..NPOL-007
- **Steps:**
  - Apply in order per namespace: `00-deny-all.yaml` → `allow-dns.yaml` → `10-allow-from-frontend.yaml`/`15-allow-from-ingress.yaml` → optional `20-allow-from-monitoring.yaml` → optional `30-allow-egress-external.yaml`.
  - Validate:
    - External: site loads 200 via Ingress; frontend→backend works; `/api/health` OK.
    - Internal: `kubectl exec` curl backend service from frontend pod succeeds; DNS resolves.
    - Observability: Prometheus targets `Up` (if NPOL-006 applied).
  - Prepare a single `kubectl delete -f infra/manifests/network-policies/...` rollback command and document it.
- **Acceptance:** With policies enforced, app continues to work; removing allows reproduces expected denials; rollback tested.

---

## Triage and Validation Tasks

### NPOL-TRIAGE-001 — Confirm enforcement engine and Kyverno relaxations

- **Dependencies:** AWSZ-006, NPOL-000
- **Steps:**
  - Confirm Calico components are Ready in `tigera-operator` and `calico-system` namespaces.
  - Confirm Kyverno ClusterPolicies in use match repo intent: `require-image-registry` allows `ghcr.io/*|quay.io/*`; `require-run-as-non-root` is `audit`; `require-resource-limits` is `enforce`.
- **Acceptance:** Output of `kubectl get pods -n tigera-operator,calico-system` shows Ready; `kubectl get cpol` shows expected policies and `validationFailureAction` values.

### NPOL-TRIAGE-002 — Verify default-deny and DNS allows are applied from repo

- **Dependencies:** NPOL-002, NPOL-003
- **Steps:**
  - Ensure these manifests are applied as-is:
    - `infra/manifests/network-policies/frontend-dev/00-deny-all.yaml`
    - `infra/manifests/network-policies/backend-dev/00-deny-all.yaml`
    - `infra/manifests/network-policies/frontend-dev/allow-dns.yaml`
    - `infra/manifests/network-policies/backend-dev/allow-dns.yaml`
- **Acceptance:** `kubectl get netpol -n frontend-dev` and `-n backend-dev` list the above policies; DNS resolution works from app pods.

### NPOL-TRIAGE-003 — Verify allow rules for backend ingress and frontend

- **Dependencies:** NPOL-004, NPOL-005
- **Steps:**
  - Confirm in `backend-dev` namespace the following are applied:
    - `infra/manifests/network-policies/backend-dev/10-allow-from-frontend.yaml` (from namespace label `name=frontend-dev` to pods `app=backend`).
    - `infra/manifests/network-policies/backend-dev/15-allow-from-ingress.yaml` (from namespace label `name=ingress` and pods labeled `app.kubernetes.io/name=ingress-nginx` to pods `app=backend`).
- **Acceptance:** `kubectl get netpol -n backend-dev` includes both; `kubectl describe netpol` shows selectors as intended.

### APP-NET-001 — Validate backend-sample Service and Deployment alignment

- **Dependencies:** NPOL-TRIAGE-003
- **Steps:**
  - Confirm `charts/backend-sample/templates/deployment.yaml` exposes `containerPort: 3001` with port name `http`.
  - Confirm `charts/backend-sample/templates/service.yaml` uses `targetPort: http` and selector matches Deployment labels.
  - In-cluster, verify endpoints map to pod IP:3001 for the Service.
- **Acceptance:** `kubectl get svc -n backend-dev` shows Service; `kubectl get endpoints <svc> -n backend-dev` shows endpoints on `:3001`.

### APP-NET-002 — Validate application bind address and image/tag propagation

- **Dependencies:** APP-NET-001
- **Steps:**
  - Ensure the backend app listens on `0.0.0.0:3001` inside container.
  - If code changed, rebuild and push image via CI; bump `charts/backend-sample/values.yaml` `image.tag` (or use immutable tags policy) so Argo deploys the new image.
  - Trigger/sync via Argo; wait for rollout to complete.
- **Acceptance:** New image tag/digest present in registry; `kubectl rollout status deploy -n backend-dev` succeeds; logs show listening on `0.0.0.0:3001`.

### GITOPS-ALIGN-001 — Argo application sanity

- **Dependencies:** AWSZ-021
- **Steps:**
  - Verify `infra/gitops/apps/*` destinations use `server: https://kubernetes.default.svc` and correct namespaces.
  - Confirm repo access secret exists/applied (e.g., `infra/gitops/repos/web-hosting-repo-secret.yaml`).
  - Ensure `root` app is applied; child apps `backend-sample` and `frontend-sample` are Healthy/Synced.
- **Acceptance:** `argocd app get root` and app children show Healthy/Synced (or equivalent `kubectl get applications.argoproj.io -n argocd`).

### NPOL-VALIDATE-001 — Functional checks under policies

- **Dependencies:** NPOL-TRIAGE-002..003, APP-NET-002, GITOPS-ALIGN-001
- **Steps:**
  - From a debug pod in `frontend-dev`, curl backend Service DNS on `/api/health`.
  - External via Ingress, curl the domain path `/api/health` (HTTP/HTTPS depending on TLS state).
- **Acceptance:** Both internal and external requests return HTTP 200.

### KYVERNO-REVERT-001 — Tighten Kyverno baseline post-install

- **Dependencies:** NPOL-VALIDATE-001
- **Steps:**
  - Revisit `infra/manifests/kyverno-policies/policies.yaml` and restore intended enforcement after Calico install and app validation.
  - Restrict allowed registries if possible (keep `quay.io` only if required by base images).
  - Apply and verify workloads remain compliant.
- **Acceptance:** `kubectl get cpol` shows intended `validationFailureAction` values; new rollouts succeed without policy violations.

### OBS-DOCS-001 — Record selectors and enforcement engine

- **Dependencies:** NPOL-TRIAGE-003
- **Steps:**
  - Update `../observability.md` to reflect final namespace labels (`name=frontend-dev|backend-dev|ingress|monitoring`) and app labels (`app=frontend|backend`).
  - Record Calico as the enforcement engine and its version.
- **Acceptance:** Documentation merged/updated; selectors align with applied NetworkPolicies.

### ROLLBACK-READY-001 — Single-command rollback plan

- **Dependencies:** NPOL-TRIAGE-002..003
- **Steps:**
  - Prepare a one-liner per namespace to delete applied NetworkPolicies under `infra/manifests/network-policies/...`.
  - Add the commands and short smoke test notes to `tasks/task_log.md`.
- **Acceptance:** `task_log.md` contains rollback commands and validated notes; dry-run confirms manifests paths resolve.
