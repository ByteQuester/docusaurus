# Task Reconciliation Status

Evidence-based status based on current repo. Categories mirror `tasks/`.

Legend: DONE = present/working evidence; TODO = not found; FIX = present but requires correction; N/A = optional/later.

## Repository and Workspace

- REPO-001 Bootstrap: DONE (structure present)
- REPO-002 Frontend integrated: DONE (`apps/frontend`)
- REPO-003 Workspaces: DONE (`pnpm-workspace.yaml` present)
- REPO-004 Lint/format baseline: DONE (eslint config package, scripts)
- REPO-005 Pre-commit hooks: TODO (no pre-commit config found)
- REPO-006 Makefile tasks: DONE (`Makefile` present)
- REPO-007 Governance: DONE (`CONTRIBUTING.md`, templates likely; verify PR/issue templates)

## CI/CD and Supply Chain

- CI-001 Bootstrap CI: DONE (`.github/workflows/ci.yml` with lint/build/test)
- CI-002 Build/push images: DONE (push on main)
- CI-003 Helm lint/template: DONE
- CI-004 Security scanning baseline: PARTIAL (jobs configured) — FIX: referenced configs missing under `policy/` (`.gitleaks.toml`, `.trivyignore`, `.checkov.yaml`)
- CI-005 Caching/perf: DONE (actions cache set)
- CI-006 Matrix builds: TODO
- CI-007 Release versioning/changelogs: TODO
- CI-008 SBOM: TODO
- CI-009 Image signing: TODO
- CI-010 Branch protections: TODO (repo setting)
- CI-011 Node version file alignment: FIX (workflow uses `.nvmrc`; repo shows `.node-version` in recent files)

## Containers and Helm

- CT-001 Containerize frontend: DONE (`apps/frontend/Dockerfile`)
- CT-002 Containerize sample backend: TODO
- HELM-001 Frontend chart: DONE (`charts/frontend`)
- HELM-002 Backend chart: DONE (`charts/backend-sample`)
- HELM-003 Publish Helm repo: TODO
- HELM-004 Chart testing automation: TODO (only lint/template present)
- HELM-005 Values management: DONE (`values-dev.yaml`, `values-prod.yaml` exist)

## Infrastructure and Cluster

- INF-001 IaC modules skeleton: DONE (`infra/modules/cluster/{aws-eks,gcp-gke}`)
- INF-002 Provision cluster: TODO (docs only)
- INF-003 Ingress controller: TODO (manifests exist; not applied)
- INF-004 cert-manager: TODO (manifests exist; not applied)
- INF-005 ExternalDNS: TODO (manifests exist; not applied)
- INF-006 DNS/domain: TODO
- INF-007 Storage class: TODO
- INF-008 Node autoscaling: N/A (optional)
- INF-009 Remote TF state: TODO (private setup; doc task)
- INF-010 Secrets baseline plan: PARTIAL (`.sops.yaml` present; ESO docs present)

## GitOps and Deployment

- GITOPS-001 Select controller: DONE (`docs/gitops.md` present, Argo CD implied)
- GITOPS-002 Install controller: TODO (manifests docs only)
- GITOPS-003 Define applications: PARTIAL (`infra/gitops/apps/frontend.yaml` present) — FIX: `destination.server` typo (`httpshttps://...`)
- GITOPS-004 Sync/health policies: PARTIAL (auto sync in file) — depends on install
- GITOPS-005 App-of-Apps: TODO
- GITOPS-006 Promotion flow: TODO
- GITOPS-007 Fix destination URL: TODO

## Security and Compliance

- SEC-001 Secret scanning in CI: PARTIAL (job exists) — depends on policy config
- SEC-002 Container/FS scanning: PARTIAL (Trivy job exists) — depends on `.trivyignore`
- SEC-003 IaC scanning: PARTIAL (Checkov job exists) — depends on `.checkov.yaml`
- SEC-004 Admission policies: TODO (kyverno manifests present; tune/apply)
- SEC-005 Pod Security Admission: TODO (document/labels)
- SEC-006 NetworkPolicies: PARTIAL (manifests present; tune namespaces)
- SEC-007 RBAC hardening: PARTIAL (rbac manifests present; tune namespaces)
- SEC-008 External Secrets Operator: TODO (manifests present; configure provider)
- SEC-009 SOPS with age: PARTIAL (`.sops.yaml` present; keys/doc needed)
- SEC-010 Image signing/verify: TODO
- SEC-011 Dependency updates: DONE (`.github/dependabot.yml` present)
- SEC-012 CDN/WAF rate limits (docs): TODO

## Observability

- OBS-001 Metrics stack: PARTIAL (manifests present; not applied)
- OBS-002 Grafana dashboards: TODO
- OBS-003 Logging (Loki): PARTIAL (manifests present; not applied)
- OBS-004 Tracing: N/A (optional)
- OBS-005 Alerts: PARTIAL (prom rules present; depends on stack)
- OBS-006 SLOs and error budgets: TODO
- OBS-007 Synthetic uptime checks: TODO

## Backend Onboarding

- BE-001 Sample backend scaffold: DONE (`apps/backend-sample` present)
- BE-002 Dockerize backend: DONE (Dockerfile present)
- BE-003 Backend Helm chart: DONE (`charts/backend-sample`)
- BE-004 Ingress + CORS: PARTIAL (values.yaml has config; needs deploy and verify)
- BE-005 OpenAPI [optional]: N/A
- BE-006 Config and flags: PARTIAL (ConfigMap values in chart; app wiring TBD)
- BE-007 Tests and quality gates: PARTIAL (`index.test.js` present; ensure CI runs)

## Docs and Portfolio

- DOC-001 Docs site scaffold: DONE (`docs/` present)
- DOC-002 Architecture diagrams: PARTIAL
- DOC-003 CI/CD overview: PARTIAL
- DOC-004 Security page: PARTIAL
- DOC-005 Observability page: DONE (plan present2)
- DOC-006 Runbooks: TODO
- DOC-007 Portfolio showpieces: PARTIAL
- DOC-008 Verification checklist: TODO

## Operational Readiness

- OPS-001 k6 baseline: DONE (`ops/k6/smoke.js`)
- OPS-002 Performance profiling: N/A
- OPS-003 Backup/restore plan: TODO
- OPS-004 Incident response: TODO
- OPS-005 Cost controls: DONE (`docs/cost-controls.md`)
- OPS-006 Release process: PARTIAL (`docs/docs/release-process.md`)
- OPS-007 Progressive delivery: N/A
- OPS-008 Autoscaling policies: PARTIAL (HPA in charts)
- OPS-009 CDN configuration: TODO
- OPS-010 SLA/SLO acceptance: TODO

## Immediate FIX items

- CI-011 — Align Node version file (.nvmrc vs .node-version)
- CI-012 — Add missing policy files referenced by CI (`policy/.gitleaks.toml`, `policy/.trivyignore`, `policy/.checkov.yaml`)
- GITOPS-007 — Fix Argo CD `destination.server` URL in `infra/gitops/apps/frontend.yaml`
