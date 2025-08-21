---
id: grafana-dashboards
slug: /reference/grafana-dashboards
sidebar_position: 10
---

# Grafana Dashboards

:::info This document provides an overview of the Grafana dashboards used in this project. :::

## Stock Dashboards

These dashboards are recommended for a baseline understanding of the cluster's health and performance. They ship with `kube-prometheus-stack` and are usually wired to the correct metrics out of the box.

- **Kubernetes / Compute Resources / Cluster:** Provides a high-level overview of the cluster's resource usage.
- **Kubernetes / Compute Resources / Nodes:** Shows detailed metrics for each node in the cluster.
- **Kubernetes / Compute Resources / Pods:** Displays resource usage for each pod in the cluster.

Record the dashboard UIDs you decide to standardize on here:

- **Cluster UID:** `<fill>`
- **Nodes UID:** `<fill>`
- **Pods UID:** `<fill>`

:::tip Accessing Grafana

- **Port-forward Grafana:** `kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80`
- The UI will be available at `http://localhost:3000`.
- Default admin credentials are set by the chart; rotate per ops policy. :::

## Imported Dashboards

:::warning These dashboards are optional and may require additional configuration to work correctly. :::

- **Kubernetes - Cluster Overview**
  - **ID:** `15757`
  - **Status:** Imported manually, currently shows no data.
  - **Notes:** Often requires query or datasource adjustments. Prefer stock dashboards until after go-live. See [Known Issues](runbooks/known-issues.md).
