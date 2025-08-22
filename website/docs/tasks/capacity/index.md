---
id: capacity-baseline
slug: /tasks/capacity/
title: 'Capacity Baseline'
---

# Capacity Baseline and Visibility

**Goal:** Establish metrics/dashboards/alerts to quantify current utilization and drive scaling decisions.

### CAP-001 — Cluster capacity and saturation dashboard

- **Dependencies**: AWSZ-024 (Prometheus stack), AWSZ-025 (uptime rules optional)
- **Steps**:
  - Import or create a Grafana dashboard showing:
    - Node CPU/memory allocatable vs used
    - Pod count vs capacity; pending pods by reason
    - Top workloads by CPU/memory
    - Cluster Autoscaler/Karpenter events (if installed)
  - Save dashboard to `docs/docs/grafana-dashboards.md` (UID/link)
- **Acceptance Criteria**: Single dashboard shows at-a-glance cluster saturation and hot workloads.

### CAP-002 — Prometheus alerts for capacity

- **Dependencies**: AWSZ-024
- **Steps**:
  - Create a `PrometheusRule` (e.g., `infra/manifests/prometheus/rules/capacity.yaml`) with alerts:
    - NodeMemoryPressure/NodeDiskPressure
    - CPU/Memory utilization > 80% for 10m
    - Pending pods for > 5m due to `insufficient cpu|memory`
  - Apply to `prometheus` namespace
- **Acceptance Criteria**: Rules load; test alert fires when threshold breached.

### CAP-003 — Requests/limits audit

- **Status**: Completed
- **Dependencies**: CAP-001
- **Steps**:
  - Use `kubectl` and/or `kubectl describe` to list Deployments without requests/limits
  - Optional: run `polaris` or `kube-score` locally to audit manifests
  - Record a list of offending workloads and file follow-up tasks
- **Acceptance Criteria**: Inventory of workloads missing requests/limits exists in `tasks/task_log.md`.
- **Audit Report**: [./audits/CAP-003_requests-limits-audit.md](./audits/CAP-003_requests-limits-audit.md)

### CAP-004 — Sizing reference capture

- **Goal**: Capture 95th percentile CPU/memory over 24h for critical workloads and record in docs.

#### CAP4-001 — Prepare Prometheus queries

- **Status**: In Progress
- **Dependencies**: CAP-001
- **Steps**:
  - Build PromQL for p95 CPU and memory per Deployment.
  - Validate in Prometheus UI; export to Grafana if desired.
- **Acceptance Criteria**: Queries return data for target workloads.

#### CAP4-002 — Record baselines

- **Status**: Planned
- **Dependencies**: CAP4-001
- **Steps**:
  - Add results to `docs/docs/observability.md` under Sizing Reference section.
- **Acceptance Criteria**: Baselines recorded to inform HPA/VPA and node sizing.
