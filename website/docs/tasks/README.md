## Task Catalog

This directory contains ordered, atomic tasks grouped by domain. Each task includes an ID, dependencies, deliverables, and acceptance criteria. Execute tasks in-file order unless dependencies indicate otherwise.

Conventions:

- IDs use a domain prefix (REPO, CI, CT, HELM, INF, GITOPS, SEC, OBS, BE, DOC, OPS) and an incremental number.
- Dependencies reference task IDs across files.
- Keep tasks small; open one PR per task.
- Never commit secrets, kubeconfigs, or Terraform state. Use placeholders for environment-specific values.

Files:

- `00_repo.md` — repository and workspace foundations
- `10_ci_cd.md` — pipelines, scanning, supply chain
- `11_ci_smart_build.md` — smart change detection and matrix builds
- `15_blueprints.md` — reusable templates for services, CI, and IRSA
- `16_monorepo_tooling.md` — evaluate/adopt Turborepo or Nx with ADR
- `17_docs_workstream.md` — documentation tasks and cheat sheets
- `20_containers_helm.md` — container images and Helm charts
- `30_infra_cluster.md` — cluster provisioning, ingress, DNS, TLS
- `40_gitops_deploy.md` — GitOps controllers and app definitions
- `50_security.md` — policies, secrets, admission, network
- `60_observability.md` — metrics, logs, traces, alerts, SLOs
- `70_backend_onboarding.md` — backend services added and exposed
- `80_docs_portfolio.md` — documentation and portfolio artifacts
- `90_operational_readiness.md` — load, backups, release, cost
- `00_aws_zero_to_hero.md` — AWS EKS end-to-end atomic tasks
