# Capacity Baseline and Visibility (Atomic Tasks)

Goal: Establish metrics/dashboards/alerts to quantify current utilization and drive scaling decisions.

## CAP-001 — Cluster capacity and saturation dashboard

- Dependencies: AWSZ-024 (Prometheus stack), AWSZ-025 (uptime rules optional)
- Steps:
  - Import or create a Grafana dashboard showing:
    - Node CPU/memory allocatable vs used
    - Pod count vs capacity; pending pods by reason
    - Top workloads by CPU/memory
    - Cluster Autoscaler/Karpenter events (if installed)
  - Save dashboard to `docs/docs/grafana-dashboards.md` (UID/link)
- Acceptance: Single dashboard shows at-a-glance cluster saturation and hot workloads.

## CAP-002 — Prometheus alerts for capacity

- Dependencies: AWSZ-024
- Steps:
  - Create a `PrometheusRule` (e.g., `infra/manifests/prometheus/rules/capacity.yaml`) with alerts:
    - NodeMemoryPressure/NodeDiskPressure
    - CPU/Memory utilization > 80% for 10m
    - Pending pods for > 5m due to `insufficient cpu|memory`
  - Apply to `prometheus` namespace
- Acceptance: Rules load; test alert fires when threshold breached.

## CAP-003 — Requests/limits audit

- **Status:** Completed
- Dependencies: CAP-001
- Steps:
  - Use `kubectl` and/or `kubectl describe` to list Deployments without requests/limits
  - Optional: run `polaris` or `kube-score` locally to audit manifests
  - Record a list of offending workloads and file follow-up tasks
- Acceptance: Inventory of workloads missing requests/limits exists in `tasks/task_log.md`.

## CAP-004 — Sizing reference capture

- Dependencies: CAP-001
- Steps:
  - For each critical workload, capture 95th percentile CPU/mem over 24h (Prometheus queries) and add to `docs/docs/observability.md`
- Acceptance: Documented baselines exist to inform HPA/VPA and node sizing.
