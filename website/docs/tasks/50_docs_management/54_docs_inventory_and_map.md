---
id: 54-docs-inventory-and-map
slug: /tasks/50-docs-management/inventory-and-map
sidebar_position: 5
---

# Docs Inventory and Navigation Map

:::info Goal Produce a complete, categorized index of all `tasks/*.md` and `docs/docs/*.md`, and publish it as a central navigation map page. This supports agents working through the entire repo without missing files. :::

### TASK-001 — Generate full inventory

- **Dependencies**: none
- **Steps** (run from repo root):
  - List all docs:
    ```bash
    find docs/docs -type f -name "*.md" | sort
    ```
  - List all tasks:
    ```bash
    find tasks -type f -name "*.md" | sort
    ```
  - Paste both lists into `docs/docs/navigation-map.md` under the respective sections (Tasks Index / Docs Index)
- **Acceptance Criteria**: Complete file lists appear in the map page.

### TASK-002 — Categorize by domain

- **Dependencies**: `DOC-MAP-001`
- **Steps**:
  - Create category groupings in `docs/docs/navigation-map.md` (e.g., GitOps/Argo, Scaling/Capacity, Observability, Security/Policies, Onboarding/Release, Infra/Cluster, CI/CD)
  - For each file in the indices, add it to one or more categories
- **Acceptance Criteria**: Every file appears in at least one category; categories are balanced and intuitive.

### TASK-003 — Key journeys and checkpoints

- **Dependencies**: `DOC-MAP-001`
- **Steps**:
  - Add "Key Journeys" sections linking the end-to-end flows:
    - **Zero-to-Hero (AWSZ-001..027)**: link to `tasks/00_aws_zero_to_hero.md` and remaining atomic tasks
    - **Scaling path**: `tasks/49_eks_scaling.md`, `tasks/51_scaling_operational_readiness.md`, `tasks/48_capacity_baseline.md`, `docs/docs/scaling.md`
    - **Observability path**: `tasks/44_prometheus_stack.md`, `tasks/45_uptime_rules.md`, `tasks/46_loki_logging.md`, `docs/docs/observability.md`
    - **Security path**: `tasks/28_network_policies_baseline.md`, `tasks/47_kyverno_baseline.md`, `docs/docs/network-policies.md`
- **Acceptance Criteria**: Journeys provide a clear “start here → next → related” view.

### TASK-004 — Cross-linking sweep tie-in

- **Dependencies**: `DOC-MAP-001`
- **Steps**:
  - Reference `tasks/53_docs_crosslinking_sweep.md` and apply its DSW-002 checklist to the Tier 1 files first
  - Add a small footer block to `docs/docs/navigation-map.md` with links to the sweep tasks and link-check instructions
- **Acceptance Criteria**: Map page guides agents to the sweep process.

### TASK-005 — Sidebar and discoverability

- **Dependencies**: none
- **Steps**:
  - Ensure `docs/docs/navigation-map.md` is visible in the sidebar (update `docs/sidebars.ts` if needed)
  - Place it near the top under "Overview" or "Index"
- **Acceptance Criteria**: Navigation map is easy to find from the docs sidebar.

### TASK-006 — Link-health check

- **Dependencies**: after initial categorization
- **Steps**:
  - Run a markdown link check over `docs/docs` and `tasks` (locally or via CI)
  - Fix any broken paths
- **Acceptance Criteria**: Link check passes cleanly.
