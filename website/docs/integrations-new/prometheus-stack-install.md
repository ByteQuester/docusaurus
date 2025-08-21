---
id: prometheus-stack-install
slug: /integrations/prometheus-stack-install
sidebar_position: 11
---

# Prometheus Stack Install Guide

:::info This document provides a step-by-step guide for installing the `kube-prometheus-stack` Helm chart. :::

## PROM-001 — Add Helm repo and update index

The first step is to add the `prometheus-community` Helm repository and update the local Helm repo index.

```bash title="Add and update Helm repo"
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

After running these commands, you can verify that the `kube-prometheus-stack` chart is available by running the following command:

```bash title="Search for kube-prometheus-stack"
helm search repo kube-prometheus-stack
```

## PROM-002 — Prepare namespace and CRDs

Next, create the `prometheus` namespace where the stack will be installed.

```bash title="Create prometheus namespace"
kubectl create namespace prometheus --dry-run=client -o yaml | kubectl apply -f -
```

:::note CRDs The `kube-prometheus-stack` chart will automatically install the necessary Custom Resource Definitions (CRDs). No further action is needed for this step. :::

## PROM-003 — Install kube-prometheus-stack with repo values

Now, install the `kube-prometheus-stack` chart using the provided values file.

```bash title="Install kube-prometheus-stack"
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -n prometheus -f infra/manifests/prometheus/values.yaml
```

After the installation is complete, you can check the status of the pods in the `prometheus` namespace:

```bash title="Check pod status"
kubectl get pods -n prometheus
```

All pods should be in the `Running` state and `Ready`.

## PROM-004 — Enable ingress-nginx ServiceMonitor

To enable Prometheus to scrape metrics from the `ingress-nginx` controller, you need to enable the `ServiceMonitor` in the `ingress-nginx` Helm chart.

Edit the `infra/manifests/ingress-nginx/values.yaml` file and add the following section:

```yaml title="infra/manifests/ingress-nginx/values.yaml"
controller:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
```

Then, upgrade the `ingress-nginx` Helm release:

```bash title="Upgrade ingress-nginx"
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx -n ingress -f infra/manifests/ingress-nginx/values.yaml
```

This will create a `ServiceMonitor` in the `ingress` namespace, which will be automatically discovered by Prometheus.

<details>
  <summary>Troubleshooting</summary>

If the `ingress-nginx` target does not appear in the Prometheus UI, you can follow these steps to troubleshoot the issue:

1.  **Check the `prometheus-operator` logs:**

    ```bash
    kubectl logs -n prometheus -l app=kube-prometheus-stack-operator
    ```

2.  **Verify that the `ServiceMonitor` is created:**

    ```bash
    kubectl get servicemonitor -n ingress ingress-nginx-controller
    ```

3.  **Check the Prometheus configuration:**
    ```bash
    kubectl exec -n prometheus prometheus-prometheus-kube-prometheus-prometheus-0 -c prometheus -- cat /etc/prometheus/config_out/prometheus.env.yaml
    ```
    This will show you the scrape configurations for Prometheus. You should see a job with the name `serviceMonitor/ingress/ingress-nginx-controller/0`.

</details>

## PROM-005 — Apply baseline alert rules (uptime)

Next, apply the baseline alert rules for uptime monitoring.

```bash title="Apply uptime rules"
kubectl apply -f infra/manifests/prometheus/rules/uptime.yaml -n prometheus
```

This will create a `PrometheusRule` resource in the `prometheus` namespace. You can verify that the rules are created by running the following command:

```bash title="Verify PrometheusRule creation"
kubectl get prometheusrule -n prometheus
```

You should see the `uptime-rules` in the list of Prometheus rules.

## PROM-006 — Verify Prometheus and Grafana

Finally, verify that Prometheus and Grafana are accessible and working correctly.

<details>
  <summary>Prometheus Verification</summary>

1.  **Port-forward the Prometheus service:**

    ```bash
    kubectl -n prometheus port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 &
    ```

2.  **Check the Prometheus targets:** Open your browser and navigate to `http://localhost:9090/targets`. You should see the `ingress-nginx`, `kube-state-metrics`, and `node-exporter` targets as `Up`.

</details>

<details>
  <summary>Grafana Verification</summary>

1.  **Port-forward the Grafana service:**

    ```bash
    kubectl -n prometheus port-forward svc/prometheus-grafana 3000:80 &
    ```

2.  **Get the Grafana admin password:**

    ```bash
    kubectl -n prometheus get secret prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d; echo
    ```

3.  **Log in to Grafana:** Open your browser and navigate to `http://localhost:3000`. Log in with the username `admin` and the password you retrieved in the previous step.

You should see the default Grafana dashboards.

</details>

## Uptime Monitoring

:::note Blackbox Exporter The `FrontendDown` and `BackendDown` alerts use the `blackbox-exporter` to probe the applications. The `blackbox-exporter` is not included in the `kube-prometheus-stack` chart, so it needs to be installed separately. :::

1.  **Install the `blackbox-exporter` Helm chart:**

    ```bash
    helm install blackbox-exporter prometheus-community/prometheus-blackbox-exporter -n prometheus
    ```

2.  **Configure Prometheus to use the `blackbox-exporter`:** Edit the `infra/manifests/prometheus/values.yaml` file and add the following `additionalScrapeConfigs` to the `prometheus.prometheusSpec` section:

    ```yaml
    additionalScrapeConfigs:
      - job_name: 'blackbox-frontend'
        metrics_path: /probe
        params:
          module: [http_2xx]
        static_configs:
          - targets:
              - http://frontend-sample.frontend-dev.svc.cluster.local
        relabel_configs:
          - source_labels: [__address__]
            target_label: __param_target
          - source_labels: [__param_target]
            target_label: instance
          - target_label: __address__
            replacement: blackbox-exporter-prometheus-blackbox-exporter:9115
      - job_name: 'blackbox-backend'
        metrics_path: /probe
        params:
          module: [http_2xx]
        static_configs:
          - targets:
              - http://backend-sample.backend-dev.svc.cluster.local/api/health
        relabel_configs:
          - source_labels: [__address__]
            target_label: __param_target
          - source_labels: [__param_target]
            target_label: instance
          - target_label: __address__
            replacement: blackbox-exporter-prometheus-blackbox-exporter:9115
    ```

3.  **Upgrade the `kube-prometheus-stack` Helm release:**
    ```bash
    helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -n prometheus -f infra/manifests/prometheus/values.yaml
    ```

### UPTIME-001 — Prerequisites

Before you can apply the uptime rules, you need to ensure that the Prometheus stack is running and that the `uptime.yaml` rules file exists.

### UPTIME-002 — Apply uptime rules

Next, apply the baseline alert rules for uptime monitoring.

```bash
kubectl apply -f infra/manifests/prometheus/rules/uptime.yaml -n prometheus
```

### UPTIME-003 — Verify rule groups loaded

<details>
  <summary>Verify Uptime Rules</summary>

To verify that the uptime rules are loaded in Prometheus, you can port-forward the Prometheus service and check the rules in the UI.

1.  **Port-forward the Prometheus service:**

    ```bash
    kubectl -n prometheus port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090 &
    ```

2.  **Check the Prometheus rules:** Open your browser and navigate to `http://localhost:9090/rules`. You should see the `uptime` rule group with the `FrontendDown` and `BackendDown` alerts.

        Alternatively, you can use the following command to check the rules from the command line:
        ```bash
        curl -s http://localhost:9090/api/v1/rules | jq '.data.groups[] | select(.name == "uptime")'
        ```

    </details>

<details>
  <summary>Troubleshooting</summary>

If the `uptime` rule group is not loaded, make sure that the `PrometheusRule` resource has the correct labels to be discovered by Prometheus. The `Prometheus` resource has a `ruleSelector` that specifies which labels to look for.

In this repository, the `Prometheus` resource selects rules with the label `release: prometheus`. You need to add this label to the `uptime-rules` `PrometheusRule` resource in the `infra/manifests/prometheus/rules/uptime.yaml` file:

```yaml
metadata:
  name: uptime-rules
  labels:
    app: kube-prometheus-stack
    release: prometheus
```

Then, apply the file again:

```bash
kubectl apply -f infra/manifests/prometheus/rules/uptime.yaml -n prometheus
```

</details>

### UPTIME-004 — (Optional) Trigger a test alert

To trigger a test alert, you can scale down a deployment to 0 replicas. For example, to trigger the `FrontendDown` alert, you can scale down the `frontend-sample` deployment:

```bash
kubectl scale deployment frontend-sample -n frontend-dev --replicas=0
```

After a minute, you should see the `TargetDown` alert firing in Prometheus. To clear the alert, scale the deployment back up:

```bash
kubectl scale deployment frontend-sample -n frontend-dev --replicas=1
```
