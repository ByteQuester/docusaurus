---
id: loki-logging
slug: /tasks/30-infra-and-testing/loki-logging
sidebar_position: 5
---

# Loki Logging

:::info **Goal:** Install and configure the Loki logging stack. :::

### LOKI-001 — Add Helm repo and update index

- **Dependencies:** none
- **Deliverables:** `grafana` Helm repository added and updated.
- **Acceptance:** `helm search repo grafana/loki-stack` shows the chart.

### LOKI-002 — Install Loki with repo values

- **Dependencies:** LOKI-001
- **Deliverables:** `loki-stack` Helm chart installed.
- **Acceptance:** All pods in the `loki` namespace are `Running` and `Ready`.

### LOKI-003 — (Optional) Install Grafana Loki datasource

- **Dependencies:** LOKI-002
- **Deliverables:** Loki datasource created in Grafana.
- **Acceptance:** Loki is available as a datasource in Grafana.

### LOKI-004 — Verify logs

- **Dependencies:** LOKI-003
- **Deliverables:** Logs are queryable in Grafana.
- **Acceptance:** Log entries are visible in the Grafana Explore UI.
