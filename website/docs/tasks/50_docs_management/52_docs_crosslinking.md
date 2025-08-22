---
id: 52-docs-crosslinking
slug: /tasks/50-docs-management/crosslinking
sidebar_position: 3
---

# Docs Cross-Linking and Navigation

:::info Goal Reduce confusion by adding consistent, minimal "Where you are / Next / Related" references across core tasks and docs. :::

### TASK-001 — Add navigation blocks to core task files

- **Dependencies**: none
- **Files**:
  - `tasks/00_aws_zero_to_hero.md`
  - `tasks/44_prometheus_stack.md`
  - `tasks/45_uptime_rules.md`
  - `tasks/46_loki_logging.md`
  - `tasks/47_kyverno_baseline.md`
  - `tasks/49_eks_scaling.md`
  - `tasks/51_scaling_operational_readiness.md`
- **Steps** (repeat per file):
  - Append a small footer with three bullets:
    - "You are here": short phrase naming the step (e.g., AWSZ-024 Prometheus stack)
    - "Next": 1-2 links to the immediate next atomic tasks
    - "Related": 1-3 links to relevant docs/runbooks
- **Acceptance Criteria**: Each listed task file ends with those three bullets and working links.

### TASK-002 — Link runbooks in troubleshooting index

- **Dependencies**: none
- **Files**:
  - Create or update `docs/docs/troubleshooting.md`
- **Steps**:
  - Add a "Runbooks" section linking to:
    - `docs/docs/argo-troubleshooting-sync-permissions.md`
    - `docs/docs/argo-troubleshooting-case-study.md`
  - Link this troubleshooting index from:
    - `docs/docs/gitops.md` (bottom "Related")
    - `docs/docs/observability.md` (bottom "Related")
- **Acceptance Criteria**: Troubleshooting index exists and is reachable from GitOps and Observability pages.

### TASK-003 — Add "Related tasks" blocks in key docs

- **Dependencies**: none
- **Files**:
  - `docs/docs/gitops.md`
  - `docs/docs/scaling.md`
  - `docs/docs/observability.md`
- **Steps**:
  - At bottom of each file, add a short "Related tasks" list linking to the most relevant task files, e.g.:
    - GitOps → `tasks/40_gitops_deploy.md`, `tasks/41_gitops_repo_access.md`, `tasks/43_gitops_sync_permissions.md`
    - Scaling → `tasks/49_eks_scaling.md`, `tasks/51_scaling_operational_readiness.md`, `tasks/48_capacity_baseline.md`
    - Observability → `tasks/44_prometheus_stack.md`, `tasks/45_uptime_rules.md`, `tasks/46_loki_logging.md`
- **Acceptance Criteria**: Each page ends with a small Related tasks list.

### TASK-004 — Roadmap index with checkpoints

- **Dependencies**: none
- **Files**:
  - Update `docs/docs/intro.md` or add `docs/docs/index-roadmap.md`
- **Steps**:
  - Add a short checklist for AWSZ-001..AWSZ-027 with current status (e.g., checked up to AWSZ-023)
  - Link to the corresponding task files for the remaining items
- **Acceptance Criteria**: Roadmap visible in sidebar; clearly shows current checkpoint and next steps.

### TASK-005 — Update AGENTS.md references

- **Dependencies**: none
- **Steps**:
  - In `AGENTS.md`, extend the "Troubleshooting references" section to also include:
    - `docs/docs/scaling.md`, `tasks/49_eks_scaling.md`, `tasks/51_scaling_operational_readiness.md`
- **Acceptance Criteria**: AGENTS.md includes links to scaling docs and tasks.

### TASK-006 — Link-health check

- **Dependencies**: after all edits above
- **Steps**:
  - Run a markdown link checker locally (or CI) against `docs/docs/` and `tasks/`
  - Fix any broken relative links
- **Acceptance Criteria**: Link checker reports no broken links.
