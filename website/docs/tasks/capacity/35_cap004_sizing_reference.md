# CAP-004 — Sizing reference capture

Goal: Capture 95th percentile CPU/memory over 24h for critical workloads and record in docs.

## CAP4-001 — Prepare Prometheus queries

- Status: In Progress
- Dependencies: CAP-001
- Steps:
  - Build PromQL for p95 CPU and memory per Deployment.
  - Validate in Prometheus UI; export to Grafana if desired.
- Acceptance: Queries return data for target workloads.

## CAP4-002 — Record baselines

- Status: Planned
- Dependencies: CAP4-001
- Steps:
  - Add results to `docs/docs/observability.md` under Sizing Reference section.
- Acceptance: Baselines recorded to inform HPA/VPA and node sizing.
