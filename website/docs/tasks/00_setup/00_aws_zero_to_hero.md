---
id: aws-zero-to-hero
slug: /tasks/00-setup/aws-zero-to-hero
sidebar_position: 1
---

# AWS Zero-to-Hero (Atomic Tasks)

:::info

End-to-end tasks to take this repo from zero to a live deployment on AWS (EKS), validated with a frontend + backend-sample and CI proofs. Each task is atomic with dependencies.

:::

**Legend:** Dependencies reference other task IDs here or in `tasks/*`.

## Phase 1: Local Setup and AWS Bootstrap

### AWSZ-001 — Prereqs and local toolchain {#awsz-001}

- **Dependencies:** none
- **Deliverables:** Local installs verified (kubectl, helm, pnpm, docker, awscli, terraform)
- **Acceptance:** `kubectl version --client`, `helm version`, `aws sts get-caller-identity` succeed

### AWSZ-002 — AWS account and IAM user/role for bootstrap {#awsz-002}

- **Dependencies:** AWSZ-001
- **Deliverables:** One of:
  - Local AWS credentials configured for bootstrap, OR
  - GitHub OIDC deploy role plan documented (to be implemented later)
- **Acceptance:** `aws sts get-caller-identity` shows correct account

### AWSZ-003 — Terraform remote state backend (private) {#awsz-003}

- **Dependencies:** AWSZ-002
- **Deliverables:** S3 bucket for tfstate and DynamoDB table for locks (names documented). Backend config not committed.
- **Acceptance:** `terraform init` uses remote state backend successfully

### AWSZ-004 — EKS Terraform variables {#awsz-004}

- **Dependencies:** AWSZ-003
- **Deliverables:** Fill placeholders for `infra/modules/cluster/aws-eks` variables (region, cluster_name, node sizes) in a private tfvars file
- **Acceptance:** `terraform validate` passes in `infra/modules/cluster/aws-eks`

## Phase 2: Cluster Provisioning and Core Components

### AWSZ-005 — Provision EKS cluster (Terraform) {#awsz-005}

- **Dependencies:** AWSZ-004
- **Deliverables:** `terraform apply` completes and outputs kubeconfig instructions
- **Acceptance:** `kubectl get nodes` shows Ready nodes

### AWSZ-006 — Kubectl context sanity checks {#awsz-006}

- **Dependencies:** AWSZ-005
- **Deliverables:** Context set to new cluster; namespace `kube-system` readable
- **Acceptance:** `kubectl config current-context` correct; `kubectl get pods -n kube-system` works

### AWSZ-007 — Ingress Controller (ingress-nginx) {#awsz-007}

- **Dependencies:** AWSZ-006
- **Deliverables:** Install ingress-nginx with repo values
- **Acceptance:** `kubectl get pods -n ingress` Ready; `kubectl get svc -n ingress` shows external LB

### AWSZ-008 — cert-manager (CRDs + controller) {#awsz-008}

- **Dependencies:** AWSZ-006
- **Deliverables:** Install cert-manager; apply `infra/manifests/cert-manager/clusterissuers.yaml` (staging first)
- **Acceptance:** `kubectl get pods -n cert-manager` Ready; `kubectl get clusterissuers` show Ready

### AWSZ-010 — Domain and DNS (manual or via ExternalDNS) {#awsz-010}

- **Dependencies:** AWSZ-007
- **Deliverables:** Route 53 hosted zone exists; `dev.<domain>` A/ALIAS to the ingress LB address (if ExternalDNS not used)
- **Acceptance:** `nslookup dev.<domain>` resolves to LB

<details>
  <summary>Optional: ExternalDNS Setup (AWSZ-009, AWSZ-009-TF, AWSZ-009-VER)</summary>

### AWSZ-009 — ExternalDNS (optional for first HTTP test) {#awsz-009}

- **Dependencies:** AWSZ-006
- **Deliverables:** Install ExternalDNS with AWS IAM policy via IRSA and `infra/manifests/external-dns/values.yaml`
- **Acceptance:** Creating an Ingress produces matching Route 53 records automatically

### AWSZ-009-TF — ExternalDNS IAM via Terraform (IRSA) {#awsz-009-tf}

- **Dependencies:** AWSZ-002, AWSZ-003
- **Deliverables:** Terraform resources for ExternalDNS IAM (policy + role + IRSA trust); Helm values updated to annotate ServiceAccount with role ARN.
- **Acceptance:** external-dns pod runs with IRSA; AWS Console shows role assumed by the pod; no credential env vars used.

### AWSZ-009-VER — Verify ExternalDNS record creation {#awsz-009-ver}

- **Dependencies:** AWSZ-009 or AWSZ-009-TF, AWSZ-010
- **Deliverables:** Create a test `Ingress` in a temporary namespace with a host under the Route 53 zone; observe ExternalDNS creates/updates the record.
- **Acceptance:** `nslookup test.<domain>` resolves to the ingress LB; delete Ingress and record is removed.

</details>

## Phase 3: CI/CD and Image Management

### AWSZ-011 — CI policy files {#awsz-011}

- **Dependencies:** CI-004
- **Deliverables:** `policy/.gitleaks.toml`, `policy/.trivyignore`, `policy/.checkov.yaml` added and tuned
- **Acceptance:** CI scanners run without missing-file errors

### AWSZ-012 — Align Node version file {#awsz-012}

- **Dependencies:** CI-001
- **Deliverables:** `.nvmrc` and/or workflow update to match Node version used locally
- **Acceptance:** CI uses intended Node version; build succeeds

### AWSZ-013 — Frontend image push {#awsz-013}

- **Dependencies:** CI-002
- **Deliverables:** Ensure GHCR push occurs on `main`; note image reference
- **Acceptance:** Image present at `ghcr.io/<owner>/<repo>:<tag>`

### AWSZ-014 — Backend-sample image push {#awsz-014}

- **Dependencies:** CI-002, BE-002
- **Deliverables:** Configure CI job or manual push for `ghcr.io/<owner>/<repo>/backend-sample:dev`
- **Acceptance:** Image present and pullable

<details>
  <summary>Optional: Sample Frontend Setup (AWSZ-012F, AWSZ-013F)</summary>

### AWSZ-012F — Sample frontend chart (optional) {#awsz-012f}

- **Dependencies:** HELM-001 (if reusing existing chart) or REPO-003 (for new chart)
- **Deliverables:** Create `charts/frontend-sample` with minimal Deployment/Service/Ingress/HPA; or reuse `charts/frontend` and parameterize for sample app.
- **Acceptance:** `helm lint` passes; `helm template` renders.

### AWSZ-013F — Sample frontend image push (optional) {#awsz-013f}

- **Dependencies:** CI-002
- **Deliverables:** Build/push `apps/frontend-sample` image to GHCR (e.g., tag `frontend-sample:dev`).
- **Acceptance:** Image present and pullable.

</details>

## Phase 4: Application Deployment

### AWSZ-015 — Frontend deploy (Helm, dev) {#awsz-015}

- **Dependencies:** AWSZ-007, AWSZ-013, HELM-001
- **Deliverables:** Deploy `charts/frontend` to namespace `frontend-dev` with values-dev and image overrides
- **Acceptance:** `kubectl get ingress -n frontend-dev` shows address; HTTP 200 at `/`

### AWSZ-016 — Backend-sample deploy (Helm, dev) {#awsz-016}

- **Dependencies:** AWSZ-007, AWSZ-014, HELM-002
- **Deliverables:** Deploy `charts/backend-sample` to namespace `backend-dev` with host set to same domain and path `/api`
- **Acceptance:** HTTP 200 at `/api/health`

### AWSZ-017 — Wire frontend → backend {#awsz-017}

- **Dependencies:** AWSZ-015, AWSZ-016
- **Deliverables:** Set `VITE_API_BASE_URL` via chart values/ConfigMap; rebuild if needed; redeploy frontend
- **Acceptance:** Frontend displays data fetched from backend

<details>
  <summary>Optional: Sample Frontend Deploy (AWSZ-015F)</summary>

### AWSZ-015F — Sample frontend deploy (Helm, dev) {#awsz-015f}

- **Dependencies:** AWSZ-007, AWSZ-013F, AWSZ-012F
- **Deliverables:** Deploy `charts/frontend-sample` (or `charts/frontend` repointed) to `frontend-dev` with image overrides.
- **Acceptance:** Ingress address responds 200 with sample page.

</details>

## Phase 5: TLS and GitOps

### AWSZ-018 — TLS staging {#awsz-018}

- **Dependencies:** AWSZ-008, AWSZ-010, AWSZ-015
- **Deliverables:** Use staging ClusterIssuer to obtain ACME staging cert
- **Acceptance:** Ingress annotated; certificate object Ready (staging issuer)

### AWSZ-019 — TLS production {#awsz-019}

- **Dependencies:** AWSZ-018
- **Deliverables:** Switch to production ClusterIssuer
- **Acceptance:** Valid HTTPS padlock at `https://dev.<domain>`

### AWSZ-020 — Argo CD install {#awsz-020}

- **Dependencies:** AWSZ-006
- **Deliverables:** Install Argo CD in `argocd` namespace; secure admin access
- **Acceptance:** `kubectl get pods -n argocd` Ready; UI reachable (port-forward ok)
  ```bash
  # port frwd
  kubectl port-forward svc/argocd-server -n argocd 8080:443 --kubeconfig $HOME/.kube/config
  # pass obtain
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" --kubeconfig $HOME/.kube/config | base64 -d
  ```

<details>
  <summary>Optional: GitOps Management (AWSZ-021, AWSZ-022, AWSZ-023)</summary>

### AWSZ-021 — Fix Argo Application destination URL {#awsz-021}

- **Dependencies:** AWSZ-020
- **Deliverables:** Update `infra/gitops/apps/frontend.yaml` `spec.destination.server` to `https://kubernetes.default.svc`
- **Acceptance:** Argo accepts the Application, showing valid destination

### AWSZ-022 — GitOps manage frontend (optional) {#awsz-022}

- **Dependencies:** AWSZ-021, GITOPS-003
- **Deliverables:** Apply Argo Application; remove manual Helm release if desired; app Healthy/Synced
- **Acceptance:** Argo sync enforces drift correction

### AWSZ-023 — GitOps app-of-apps and backend app (optional) {#awsz-023}

- **Dependencies:** AWSZ-022
- **Deliverables:** Apply `infra/gitops/apps/root.yaml` and `backend.yaml` to include backend-sample
- **Acceptance:** Both apps Healthy/Synced under Argo

</details>

## Phase 6: Observability and Security

### AWSZ-024 — Prometheus stack {#awsz-024}

- **Dependencies:** AWSZ-006
- **Deliverables:** Install kube-prometheus-stack with `infra/manifests/prometheus/values.yaml`
- **Acceptance:** Prometheus and Grafana pods Ready; UI accessible via port-forward

### AWSZ-025 — Uptime rules {#awsz-025}

- **Dependencies:** AWSZ-024
- **Deliverables:** Apply `infra/manifests/prometheus/rules/uptime.yaml`
- **Acceptance:** Rules created; alerts visible in Prometheus/Alertmanager

### AWSZ-027 — Kyverno policies (baseline) {#awsz-027}

- **Dependencies:** AWSZ-006
- **Deliverables:** Apply `infra/manifests/kyverno-policies/policies.yaml` (tune namespaces)
- **Acceptance:** Non-compliant pods denied; compliant pods run

### AWSZ-028 — NetworkPolicies (baseline) {#awsz-028}

- **Dependencies:** AWSZ-006, AWSZ-015, AWSZ-016
- **Deliverables:** Apply `infra/manifests/network-policies/policies.yaml` with frontend↔backend allows
- **Acceptance:** Traffic blocked until allow rules applied; `/api/health` recovers with policy

### AWSZ-029 — RBAC for apps {#awsz-029}

- **Dependencies:** AWSZ-006
- **Deliverables:** Apply `infra/manifests/rbac/{frontend-rbac.yaml,backend-rbac.yaml}` and bind to ServiceAccounts
- **Acceptance:** App pods run with least privilege; no cluster-wide list permissions

<details>
  <summary>Optional: Loki Logging (AWSZ-026)</summary>

### AWSZ-026 — Loki logging (optional) {#awsz-026}

- **Dependencies:** AWSZ-006
- **Deliverables:** Install Loki per `infra/manifests/loki/values.yaml`
- **Acceptance:** Logs queryable in Grafana

</details>

## Phase 7: Testing and Documentation

### AWSZ-030 — k6 baseline {#awsz-030}

- **Dependencies:** AWSZ-017
- **Deliverables:** Run `ops/k6/smoke.js` against live domain; record results in docs
- **Acceptance:** Latency and error thresholds met

### AWSZ-031 — Portfolio links {#awsz-031}

- **Dependencies:** AWSZ-017
- **Deliverables:** Update `docs/docs/integration-links.md` with live site, `/api/health`, GHCR image URIs; add screenshots
- **Acceptance:** Docs build and links render

### AWSZ-032 — Release process doc alignment {#awsz-032}

- **Dependencies:** CI-007 (when implemented)
- **Deliverables:** Update `docs/docs/release-process.md` and promotion flow with current practice
- **Acceptance:** Docs reflect reality; example release created

### AWSZ-033 — Cleanup duplicate frontend under env (if any) {#awsz-033}

- **Dependencies:** REPO-002
- **Deliverables:** Remove `env/dev/src/frontend/*` code if still present; leave README pointer
- **Acceptance:** Single source of truth at `apps/frontend`

### AWSZ-034 — Final verification checklist {#awsz-034}

- **Dependencies:** All above
- **Deliverables:** Run through `plans/validation.md`; mark each check pass/fail with notes
- **Acceptance:** All mandatory checks pass; issues tracked as tasks

---

:::tip Notes

- For speed, ExternalDNS (AWSZ-009) can be deferred; set DNS manually for first TLS.
- Prefer IRSA for controllers needing AWS access (ExternalDNS, cert-manager DNS01). :::
