---
id: uptime-rules
slug: /tasks/30-infra-and-testing/uptime-rules
sidebar_position: 4
---

# Uptime Rules

:::info **Goal:** Apply and verify baseline alert rules for uptime monitoring. :::

### UPTIME-001 — Prerequisites

- **Dependencies:** PROM-003
- **Deliverables:** Prometheus stack running; `uptime.yaml` rules file exists.
- **Acceptance:** Prometheus pods are `Running` and `Ready`; `uptime.yaml` file is present.

### UPTIME-002 — Apply uptime rules

- **Dependencies:** UPTIME-001
- **Deliverables:** `PrometheusRule` resource created in the `prometheus` namespace.
- **Acceptance:** `kubectl get prometheusrule -n prometheus` shows the `uptime-rules`.

### UPTIME-003 — Verify rule groups loaded

- **Dependencies:** UPTIME-002
- **Deliverables:** Uptime rules loaded in Prometheus.
- **Acceptance:** `uptime` rule group with `FrontendDown` and `BackendDown` alerts is visible in the Prometheus UI.

### UPTIME-004 — (Optional) Trigger a test alert

- **Dependencies:** UPTIME-003
- **Deliverables:** Test alert triggered and cleared.
- **Acceptance:** `TargetDown` alert fires in Prometheus when a deployment is scaled down, and clears when scaled back up.
