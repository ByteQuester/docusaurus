# Operational Readiness

## OPS-001 — Load testing baseline (k6)

- Dependencies: BE-004, HELM-001
- Deliverables: `ops/k6/` smoke and baseline tests; thresholds documented.
- Acceptance: test runs and reports latency/error metrics.

## OPS-002 — Performance profiling [optional]

- Dependencies: OPS-001
- Deliverables: simple profiling of backend under load; bottlenecks noted.
- Acceptance: report included in docs.

## OPS-003 — Backup and restore plan

- Dependencies: INF-002
- Deliverables: Velero setup (if persistent state exists) or rationale why not; doc.
- Acceptance: backup and restore tested (or documented N/A).

## OPS-004 — Incident response and on-call

- Dependencies: OBS-005
- Deliverables: incident checklist, severity levels, comms template.
- Acceptance: tabletop exercise notes in docs.

## OPS-005 — Cost controls

- Dependencies: INF-002
- Deliverables: sizing guidance, autoscaling limits, schedule/off-hour policies; optional Kubecost.
- Acceptance: cost guardrails documented; optional dashboard working.

## OPS-006 — Release process

- Dependencies: CI-007
- Deliverables: semantic versioning, changelogs, promotion flow; rollback procedure.
- Acceptance: documented and exercised once.

## OPS-007 — Progressive delivery [optional]

- Dependencies: GITOPS-002
- Deliverables: Argo Rollouts or similar; canary example for backend.
- Acceptance: canary shift observed in test.

## OPS-008 — Autoscaling policies

- Dependencies: HELM-001, HELM-002
- Deliverables: HPA for services (CPU); optional KEDA for events/queue.
- Acceptance: scale up/down observed with load.

## OPS-009 — CDN configuration

- Dependencies: INF-006
- Deliverables: CDN (Cloudflare) in front of site; caching rules; basic WAF/rate limit.
- Acceptance: CDN headers present; rate limit tested.

## OPS-010 — SLA/SLO acceptance

- Dependencies: OBS-006
- Deliverables: explicit SLOs and an acceptance gate before major releases.
- Acceptance: SLO dashboard green before deploy.
