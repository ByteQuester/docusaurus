---
id: prometheus-stack
slug: /tasks/30-infra-and-testing/prometheus-stack
sidebar_position: 3
---

# Prometheus Stack

:::info **Goal:** Install and configure the Prometheus monitoring stack. :::

### PROM-001 — Add Helm repo and update index

- **Dependencies:** none
- **Deliverables:** `prometheus-community` Helm repository added and updated.
- **Acceptance:** `helm search repo kube-prometheus-stack` shows the chart.

### PROM-002 — Prepare namespace and CRDs

- **Dependencies:** none
- **Deliverables:** `prometheus` namespace created.
- **Acceptance:** `kubectl get namespace prometheus` returns the namespace.

### PROM-003 — Install kube-prometheus-stack with repo values

- **Dependencies:** PROM-002
- **Deliverables:** `kube-prometheus-stack` Helm chart installed.
- **Acceptance:** All pods in the `prometheus` namespace are `Running` and `Ready`.

### PROM-004 — Enable ingress-nginx ServiceMonitor

- **Dependencies:** PROM-003
- **Deliverables:** `ingress-nginx` Helm release upgraded with `ServiceMonitor` enabled.
- **Acceptance:** `ingress-nginx` target appears as `Up` in the Prometheus UI.
