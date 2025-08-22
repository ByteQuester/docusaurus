---
id: 90-operational-readiness
slug: /tasks/90-operational-readiness/overview
sidebar_position: 1
---

# Operational Readiness

### TASK-001 — Load testing baseline (k6)

- **Dependencies**: `BE-004`, `HELM-001`
- **Deliverables**: `ops/k6/` smoke and baseline tests; thresholds documented.
- **Acceptance Criteria**: test runs and reports latency/error metrics.

### TASK-002 — Performance profiling [optional]

- **Dependencies**: `OPS-001`
- **Deliverables**: simple profiling of backend under load; bottlenecks noted.
- **Acceptance Criteria**: report included in docs.

### TASK-003 — Backup and restore plan

- **Dependencies**: `INF-002`
- **Deliverables**: Velero setup (if persistent state exists) or rationale why not; doc.
- **Acceptance Criteria**: backup and restore tested (or documented N/A).

### TASK-004 — Incident response and on-call

- **Dependencies**: `OBS-005`
- **Deliverables**: incident checklist, severity levels, comms template.
- **Acceptance Criteria**: tabletop exercise notes in docs.

### TASK-005 — Cost controls

- **Dependencies**: `INF-002`
- **Deliverables**: sizing guidance, autoscaling limits, schedule/off-hour policies; optional Kubecost.
- **Acceptance Criteria**: cost guardrails documented; optional dashboard working.

### TASK-006 — Release process

- **Dependencies**: `CI-007`
- **Deliverables**: semantic versioning, changelogs, promotion flow; rollback procedure.
- **Acceptance Criteria**: documented and exercised once.

### TASK-007 — Progressive delivery [optional]

- **Dependencies**: `GITOPS-002`
- **Deliverables**: Argo Rollouts or similar; canary example for backend.
- **Acceptance Criteria**: canary shift observed in test.

### TASK-008 — Autoscaling policies

- **Dependencies**: `HELM-001`, `HELM-002`
- **Deliverables**: HPA for services (CPU); optional KEDA for events/queue.
- **Acceptance Criteria**: scale up/down observed with load.

### TASK-009 — CDN configuration

- **Dependencies**: `INF-006`
- **Deliverables**: CDN (Cloudflare) in front of site; caching rules; basic WAF/rate limit.
- **Acceptance Criteria**: CDN headers present; rate limit tested.

### TASK-010 — SLA/SLO acceptance

- **Dependencies**: `OBS-006`
- **Deliverables**: explicit SLOs and an acceptance gate before major releases.
- **Acceptance Criteria**: SLO dashboard green before deploy.
