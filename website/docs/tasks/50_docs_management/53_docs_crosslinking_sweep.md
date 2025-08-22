---
id: 53-docs-crosslinking-sweep
slug: /tasks/50-docs-management/crosslinking-sweep
sidebar_position: 4
---

# Docs Cross-Linking Sweep

:::info Goal Systematically add consistent navigation/cross-links and fix link hygiene across all docs/tasks by sweeping the repo in prioritized chunks. :::

### TASK-001 — Inventory and prioritization

- **Dependencies**: none
- **Steps**:
  - Generate an inventory (paste results into PR) using:
    ```bash title="Generate Inventory"
    find docs/docs -type f -name "*.md" | sort
    find tasks -type f -name "*.md" | sort
    ```
  - Prioritize into tiers:
    - **Tier 1 (critical)**: `docs/docs/gitops*.md`, `docs/docs/argo*.md`, `docs/docs/scaling.md`, `docs/docs/observability*.md`, `tasks/00_aws_zero_to_hero.md`, `tasks/40*..49*`
    - **Tier 2 (security + onboarding)**: `docs/docs/security*.md`, `docs/docs/network-*.md`, `docs/docs/service-onboarding.md`, `tasks/28_network_policies_baseline.md`, `tasks/47_kyverno_baseline.md`
    - **Tier 3 (remaining)**: all others
- **Acceptance Criteria**: Inventory attached to PR; tiers labeled.

### TASK-002 — Per-file checklist (apply to every file in a chunk)

- **Dependencies**: none
- **Steps** (repeat per file):
  - Ensure frontmatter exists (Docusaurus-friendly) where relevant
  - Add a footer block with:
    - "You are here": succinct description of the file’s purpose
    - "Next": 1–2 next logical links (task or doc)
    - "Related": up to 3 links to supporting docs/runbooks/tasks
  - Normalize headings (##/###), code fences, and backticks for file/dir names
  - Fix relative links to use correct paths; avoid bare URLs without backticks or anchors
- **Acceptance Criteria**: File renders; footer present; links valid.

### TASK-003 — Tracking and review gates

- **Dependencies**: none
- **Steps**:
  - For each chunk, add a section in `tasks/task_log.md` noting:
    - Files touched, PR link, reviewer
    - Outstanding follow-ups (e.g., missing target docs)
  - Use PR labels: `docs-sweep`, `chunk:<name>`, `tier:<1|2|3>`
- **Acceptance Criteria**: task_log.md updated per chunk; PR labeled.

### TASK-004 — Chunk A: Core tasks (Tier 1)

- **Scope**: `tasks/00_aws_zero_to_hero.md`, `tasks/40_gitops_deploy.md`, `tasks/41_gitops_repo_access.md`, `tasks/42_registry_image_pull.md`, `tasks/43_gitops_sync_permissions.md`, `tasks/44_prometheus_stack.md`, `tasks/45_uptime_rules.md`, `tasks/46_loki_logging.md`, `tasks/47_kyverno_baseline.md`, `tasks/49_eks_scaling.md`, `tasks/51_scaling_operational_readiness.md`, `tasks/52_docs_crosslinking.md`
- **Steps**:
  - Apply DSW-002 checklist to each file
- **Acceptance Criteria**: All files have consistent footer and working links.

### TASK-005 — Chunk B: GitOps & Argo docs (Tier 1)

- **Scope**: `docs/docs/gitops.md`, `docs/docs/argo-quickstart.md`, `docs/docs/argo-troubleshooting-sync-permissions.md`, `docs/docs/argo-troubleshooting-case-study.md`
- **Steps**:
  - Apply DSW-002
  - Ensure cross-links among GitOps intro, quickstart, and runbooks
- **Acceptance Criteria**: GitOps/Argo docs form a navigable cluster.

### TASK-006 — Chunk C: Scaling & Observability (Tier 1)

- **Scope**: `docs/docs/scaling.md`, `docs/docs/observability.md`, `docs/docs/grafana-dashboards.md`, `docs/docs/verification-checklist.md`
- **Steps**:
  - Apply DSW-002; link to tasks `49`, `51`, `44`, `45`, `46`
- **Acceptance Criteria**: Clear path from strategy → tasks → dashboards.

### TASK-007 — Chunk D: Security & Policies (Tier 2)

- **Scope**: `docs/docs/security.md`, `docs/docs/network-policies.md`, `docs/docs/pod-security-admission.md`, `tasks/28_network_policies_baseline.md`, `tasks/47_kyverno_baseline.md`
- **Steps**:
  - Apply DSW-002; cross-link policies to manifests in `infra/manifests/kyverno-policies/`
- **Acceptance Criteria**: Security docs point to applied policies and tasks.

### TASK-008 — Chunk E: Onboarding & Release (Tier 2)

- **Scope**: `docs/docs/service-onboarding.md`, `docs/docs/release-process.md`, `docs/docs/promotion-flow.md`
- **Steps**:
  - Apply DSW-002; link to GitOps tasks and templates under `infra/gitops/apps/_template-app.yaml`
- **Acceptance Criteria**: New service path is discoverable and complete.

### TASK-009 — Chunk F: Remaining docs (Tier 3)

- **Scope**: all remaining `docs/docs/*.md`
- **Steps**:
  - Apply DSW-002
- **Acceptance Criteria**: Consistent footers and fixed links repo-wide.

### TASK-010 — Link check and completion

- **Dependencies**: DSW-004..009
- **Steps**:
  - Run a local link checker across `docs/docs` and `tasks`
  - Fix any broken links found; re-run until clean
- **Acceptance Criteria**: No broken links reported; sweep considered complete.
