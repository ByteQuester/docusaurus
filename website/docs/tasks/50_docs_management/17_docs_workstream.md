---
id: 17-docs-workstream
slug: /tasks/50-docs-management/workstream
sidebar_position: 1
---

# Documentation Workstream

:::info Goal Turn the work done into clear, actionable docs and cheat sheets. :::

### TASK-101 — CI smart build doc

- **Dependencies**: `CI-013`..`CI-017`
- **Deliverables**: `docs/docs/ci-smart-build.md` explaining change detection, matrices, decoupled helm test, scanners, with code excerpts.
- **Acceptance Criteria**: Page renders, links to `.github/workflows/ci.yml` anchors.

### TASK-102 — Infra provisioning and IRSA

- **Dependencies**: `AWSZ-003`..`AWSZ-009-TF`
- **Deliverables**: Extend `docs/docs/dns-tls.md` and add `docs/docs/irsa-externaldns.md` capturing the IRSA module usage, values.yaml annotations, and verification steps.
- **Acceptance Criteria**: Docs show commands used and expected outputs; link to module README.

### TASK-103 — Service onboarding cheat sheet

- **Dependencies**: `BP-001`..`BP-004`
- **Deliverables**: `docs/docs/service-onboarding.md` with a numbered checklist: create app, chart from template, CI caller, values-dev/prod, ingress path, CORS.
- **Acceptance Criteria**: Following steps reproduces 1onboarding for a new service.

### TASK-104 — DNS and TLS checklist

- **Dependencies**: `AWSZ-010`, `AWSZ-018`..`AWSZ-019`
- **Deliverables**: A compact checklist with Route 53 zone, A/ALIAS, staging→prod issuer switch, and quick curl/openssl tests.
- **Acceptance Criteria**: New deploy achieves TLS padlock by following it.

### TASK-105 — GitOps quickstart

- **Dependencies**: `GITOPS-001`..`GITOPS-003`, `GITOPS-007`
- **Deliverables**: `docs/docs/gitops-quickstart.md` with install steps, Application template usage, and drift demo.
- **Acceptance Criteria**: Running through steps yields a Healthy/Synced app in Argo.

### TASK-106 — Observability quickstart

- **Dependencies**: `OBS-001`..`OBS-005`
- **Deliverables**: `docs/docs/observability-quickstart.md` for kube-prometheus-stack install, port-forward, dashboard list, uptime rules apply.
- **Acceptance Criteria**: Operator can view CPU/mem/ingress and uptime alerts after following it.

### TASK-107 — Verification diary

- **Dependencies**: `plans/validation.md`
- **Deliverables**: For each proof, paste the real commands/outputs/URLs into `docs/docs/integration-links.md` or a new `verification-diary.md`.
- **Acceptance Criteria**: Artifacts are present for portfolio reviewers.

### TASK-108 — Argo CD install and usage

- **Dependencies**: `AWSZ-020`..`AWSZ-022`, `GITOPS-003`, `GITOPS-007`
- **Deliverables**: `docs/docs/argo-quickstart.md` with install, initial admin password retrieval, port-forward/login, Application creation (from template), and drift/self-heal demo.
- **Acceptance Criteria**: Following steps yields Healthy/Synced app and a demonstrated drift correction.

### TASK-109 — Ingress-NGINX setup

- **Dependencies**: `AWSZ-007`
- **Deliverables**: `docs/docs/ingress-nginx.md` covering Helm install command, values used, verifying controller Ready, obtaining LB address, and basic troubleshooting.
- **Acceptance Criteria**: A new operator can replicate install and retrieve ingress address.

### TASK-110 — ExternalDNS setup

- **Dependencies**: `AWSZ-009` or `AWSZ-009-TF`, `AWSZ-009-VER`, `AWSZ-010`
- **Deliverables**: `docs/docs/externaldns.md` explaining IRSA role (link to module), domainFilters, values.yaml, verification with test Ingress, and cleanup.
- **Acceptance Criteria**: Test host resolves via Route 53 and is removed on Ingress deletion.

### TASK-111 — EKS Terraform usage

- **Dependencies**: `AWSZ-003`..`AWSZ-006`
- **Deliverables**: `docs/docs/eks-terraform.md` with backend init (S3+DynamoDB), tfvars shape (redacted), apply/destroy commands, and common pitfalls (public IP, routes, node group).
- **Acceptance Criteria**: New operator provisions a working cluster by following it.

### TASK-112 — Helm charts overview

- **Dependencies**: `HELM-001`..`HELM-005`
- **Deliverables**: `docs/docs/helm-charts.md` describing chart structure, values-dev/prod patterns, common annotations (ingress class, cert-manager issuer), and how to override image/tag.
- **Acceptance Criteria**: Operator can deploy either frontend/backend chart with correct values.

### TASK-113 — Frontend ↔ backend wiring

- **Dependencies**: `AWSZ-017`
- **Deliverables**: `docs/docs/wiring-frontend-backend.md` describing `VITE_API_BASE_URL` (or equivalent), where to set it (ConfigMap/build-time), and how to verify in UI.
- **Acceptance Criteria**: Page shows a visible data fetch and how to change the endpoint.

### TASK-114 — App-of-apps GitOps pattern

- **Dependencies**: `AWSZ-023`, `GITOPS-005`
- **Deliverables**: `docs/docs/gitops-app-of-apps.md` showing the root Application referencing child apps; how to add a new service Application.
- **Acceptance Criteria**: New service appears managed under Argo via documented steps.

### TASK-115 — RBAC, PSA, and NetworkPolicies

- **Dependencies**: `SEC-005`..`SEC-007`, `SEC-006`
- **Deliverables**: `docs/docs/security-namespaces.md` explaining namespace labels for PSA, minimal ServiceAccounts/Roles, and baseline NetworkPolicies (default-deny + allow lists).
- **Acceptance Criteria**: After applying examples, non-compliant pods are denied; allowed traffic flows only.

### TASK-116 — Security scanners configuration

- **Dependencies**: `CI-004`, `CI-012`, `SEC-001`..`SEC-003`
- **Deliverables**: `docs/docs/security-scanning.md` documenting `.gitleaks.toml`, `.trivyignore`, `.checkov.yaml` structure, thresholds, and how to interpret SARIF in GitHub Security.
- **Acceptance Criteria**: Scanners run cleanly with tuned policies; page links to config files.

### TASK-117 — Release and promotion

- **Dependencies**: `CI-007`..`CI-009`, `AWSZ-032`
- **Deliverables**: Extend `docs/docs/release-process.md` to include tag-to-release flow, Helm chart version bump, SBOM attach, and promotion from dev→prod values.
- **Acceptance Criteria**: A release run matches documentation and artifacts are present.

### TASK-118 — Service template and reusable CI

- **Dependencies**: `BP-001`..`BP-004`, `CI-019`
- **Deliverables**: `docs/docs/service-template.md` and `docs/docs/ci-reusable.md` showing how to create a new service from the template and wire the reusable build/push + helm-validate workflows.
- **Acceptance Criteria**: A new service is onboarded using only the templates and docs.

### TASK-119 — Observability with uptime rules

- **Dependencies**: `OBS-001`..`OBS-005`
- **Deliverables**: Extend `docs/docs/observability-quickstart.md` with `infra/manifests/prometheus/rules/uptime.yaml` usage and how to view alerts.
- **Acceptance Criteria**: An induced outage triggers an alert as documented.

### TASK-120 — Troubleshooting index

- **Dependencies**: multiple
- **Deliverables**: `docs/docs/troubleshooting.md` linking to TLS, GitOps, Ingress, ExternalDNS, and CI sections; add quick symptom→page pointers.
- **Acceptance Criteria**: Index exists and links work.

### TASK-121: GitOps structure

- **Deliverables**: page documenting infra/gitops/apps, infra/gitops/projects, and the app-of-apps pattern.
