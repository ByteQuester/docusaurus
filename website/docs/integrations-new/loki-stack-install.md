---
id: loki-stack-install
slug: /integrations/loki-stack-install
sidebar_position: 10
---

# Loki Stack Install Guide

:::info This document provides a step-by-step guide for installing the `loki-stack` Helm chart. :::

## LOKI-001 — Add Helm repo and update index

The first step is to add the `grafana` Helm repository and update the local Helm repo index.

```bash title="Add and update Helm repo"
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

After running these commands, you can verify that the `loki-stack` chart is available by running the following command:

```bash title="Search for loki-stack"
helm search repo grafana/loki-stack
```

You should see output similar to this:

```
NAME                  	CHART VERSION	APP VERSION	DESCRIPTION
grafana/loki-stack    	2.10.2       	v2.9.3     	Loki: like Prometheus, but for logs.
```

## LOKI-002 — Install Loki with repo values

Now, install the `loki-stack` chart using the provided values file.

First, create a `values.yaml` file under `infra/manifests/loki/values.yaml`:

```yaml title="infra/manifests/loki/values.yaml"
loki:
  storage:
    type: 'filesystem'
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 200m
      memory: 256Mi
promtail:
  enabled: true
```

Then, install/upgrade the chart:

```bash title="Install loki-stack"
helm upgrade --install loki grafana/loki-stack -n loki --create-namespace -f infra/manifests/loki/values.yaml
```

After the installation is complete, you can check the status of the pods in the `loki` namespace:

```bash title="Check pod status"
kubectl get pods -n loki
```

All pods should be in the `Running` state and `Ready`.

### Troubleshooting

:::tip Troubleshooting Pending Pods If the `loki-0` pod is in a `Pending` state, you can follow these steps to troubleshoot the issue. This is often caused by insufficient cluster resources. :::

1.  **Check the pod description:**

    ```bash
    kubectl describe pod loki-0 -n loki
    ```

    If the event message is `0/2 nodes are available: 2 Too many pods`, it means that the cluster is at its pod capacity.

2.  **Check the node status:** To confirm that the nodes are at their pod capacity, you can run `kubectl describe nodes`. In the output, you will see the `Allocated resources` section for each node. If the number of pods is equal to the `pods` capacity, then the node cannot schedule new pods.

3.  **Scale down non-essential deployments:** To free up resources, you can scale down non-essential deployments. First, list all the pods in the cluster to identify which deployments can be scaled down:

    ```bash
    kubectl get pods --all-namespaces
    ```

    In our case, we identified that the `frontend`, `backend-sample`, and `frontend-sample` deployments could be scaled down. The `frontend` pods were in an `ImagePullBackOff` state, so they were not running anyway.

    Use the following commands to scale down the deployments:

    ```bash
    kubectl scale deployment frontend -n frontend-dev --replicas=0
    kubectl scale deployment backend-sample -n backend-dev --replicas=0
    kubectl scale deployment frontend-sample -n frontend-dev --replicas=0
    ```

    After scaling down the deployments, the `loki-0` pod should be scheduled and running.

## LOKI-003 — (Optional) Install Grafana Loki datasource

:::note To add Loki as a datasource in Grafana, you can use the Grafana API. :::

1.  **Port-forward the Grafana service:**

    ```bash
    kubectl -n prometheus port-forward svc/prometheus-grafana 3000:80 &
    ```

2.  **Get the Grafana admin password:**

    ```bash
    kubectl -n prometheus get secret prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d; echo
    ```

3.  **Create the Loki datasource:**
    ```bash
    curl -X POST -u admin:<grafana-password> -H "Content-Type: application/json" -d '{
      "name": "Loki",
      "type": "loki",
      "url": "http://loki.loki.svc.cluster.local:3100",
      "access": "proxy",
      "basicAuth": false
    }' http://localhost:3000/api/datasources
    ```

## LOKI-004 — Verify logs

To verify that logs are being collected, you can use the Grafana Explore UI or the Grafana API.

To use the API, you can run the following command:

```bash
curl -G -u admin:<grafana-password> http://localhost:3000/api/datasources/proxy/<datasource-id>/loki/api/v1/query_range --data-urlencode 'query={namespace="loki"}' --data-urlencode "start=<start-timestamp>" --data-urlencode "end=<end-timestamp>"
```

Replace `<grafana-password>`, `<datasource-id>`, `<start-timestamp>`, and `<end-timestamp>` with the appropriate values. You should see a JSON response with the log entries.
