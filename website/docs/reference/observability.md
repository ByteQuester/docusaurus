---
id: observability
slug: /reference/observability
sidebar_position: 15
---

# Observability Plan

:::info This document outlines the plan for setting up an observability stack for this project. Our observability strategy is based on the popular and powerful combination of Prometheus, Grafana, Loki, and Tempo (the "Grafana Stack"). :::

## Overview

Our observability stack will consist of the following components:

- **Metrics:** [Prometheus](https://prometheus.io/) for collecting and storing metrics, and [Grafana](https://grafana.com/) for visualizing them.
- **Logs:** [Loki](https://grafana.com/oss/loki/) for collecting and aggregating logs.
- **Traces:** [Tempo](https://grafana.com/oss/tempo/) for distributed tracing, with application instrumentation using [OpenTelemetry (OTel)](https://opentelemetry.io/).

This stack provides a comprehensive and cost-effective solution for monitoring our application and infrastructure.

## Metrics (Prometheus + Grafana)

We will use the `kube-prometheus-stack` Helm chart to deploy a full-featured Prometheus and Grafana setup. This chart includes:

- Prometheus Operator
- Prometheus instances
- Alertmanager
- Grafana
- A set of default dashboards and alerting rules

This will provide us with a solid foundation for monitoring our Kubernetes cluster and applications.

## Logs (Loki)

We will use Loki for log aggregation. Loki is designed to be cost-effective and easy to operate, and it integrates seamlessly with Grafana, allowing us to correlate logs with metrics.

We will use the `loki-stack` Helm chart to deploy Loki and Promtail (the agent that collects logs and sends them to Loki).

## Traces (Tempo + OpenTelemetry)

For distributed tracing, we will use Tempo. Tempo is a high-scale, minimal-dependency distributed tracing backend that also integrates perfectly with Grafana.

Our application code will be instrumented using the OpenTelemetry (OTel) libraries. OTel is an open-source observability framework that provides a standardized way to generate and collect telemetry data (traces, metrics, and logs).

We will use the `tempo` Helm chart to deploy Tempo.

## Helm Chart Dependencies

To implement this observability stack, we will use the following Helm charts:

- **kube-prometheus-stack:** For Prometheus and Grafana.
- **loki-stack:** For Loki and Promtail.
- **tempo:** For Tempo.

These charts will be added as dependencies to our main cluster configuration or deployed separately.

:::note Follow-up Tickets The actual implementation of this observability stack will be done in future tickets, once the Kubernetes cluster is up and running. Here are the placeholder tickets for the implementation:

- **OBS-1:** Deploy the `kube-prometheus-stack` Helm chart.
- **OBS-2:** Deploy the `loki-stack` Helm chart.
- **OBS-3:** Deploy the `tempo` Helm chart.
- **OBS-4:** Instrument the frontend application with OpenTelemetry. :::

:::note Implementation For a step-by-step guide on how to install the `kube-prometheus-stack`, see the [Prometheus Stack Install Guide](../integrations/prometheus-stack-install.md). :::

<details>
  <summary>Quick metrics triage (Grafana shows "No data")</summary>

Use this 5-minute checklist before deeper debugging:

1. Verify monitoring stack pods are Ready (adjust namespace if different):
   - `kubectl get pods -n monitoring | egrep \'grafana|prometheus|kube-state|node-exporter\'`
2. Check Prometheus targets are Up:
   - Port-forward Prometheus: `kubectl port-forward -n monitoring svc/kube-prometheus-stack-prometheus 9090:9090`
   - Open `http://localhost:9090/targets` and confirm key targets: `kubelet`, `cadvisor`, `kube-state-metrics`, `apiserver`, `node-exporter`.
3. Verify Grafana datasource resolves:
   - Port-forward Grafana: `kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80`
   - In Grafana → Data Sources → Prometheus, the URL should point to the in-cluster service (e.g., `http://kube-prometheus-stack-prometheus:9090`).
4. Confirm metrics-server works for k8s resource metrics:
   - `kubectl top nodes`
   - `kubectl top pods -A | head -20`
5. Ensure ServiceMonitor/PodMonitor CRDs and label selectors exist:
   - `kubectl get servicemonitors,podmonitors -A`

If targets are Up and Grafana is connected but dashboards are empty, try stock dashboards: "Kubernetes / Compute Resources / Cluster", "Nodes", "Pods". Record chosen dashboard UIDs in [Grafana Dashboards](grafana-dashboards.md). Defer custom query rewrites until after go-live.

</details>
