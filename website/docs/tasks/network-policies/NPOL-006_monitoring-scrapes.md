# NPOL-006 — Allow monitoring scrapes (optional)

- Status: Completed
- Dependencies: NPOL-002, AWSZ-024

## Scope

Permit Prometheus scrapes from `monitoring` namespace to metrics endpoints of selected pods.

## Steps

We first checked for `ServiceMonitor` resources for the `frontend-sample` and `backend-sample` applications and found that none exist. Therefore, no network policies are needed for them at this time.

We then investigated the `ingress-nginx-controller` and found that it exposes metrics on port `10254`. We created a network policy to allow ingress from the `prometheus` namespace (which has the label `name=monitoring`) to the `ingress-nginx-controller` pods on port `10254`.

**Policy:** `infra/manifests/network-policies/ingress/allow-from-prometheus.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-prometheus
  namespace: ingress
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: monitoring
      ports:
        - protocol: TCP
          port: 10254
```

**Apply command:**

```bash
kubectl apply -f infra/manifests/network-policies/ingress/allow-from-prometheus.yaml --kubeconfig=$HOME/.kube/config
```

## Acceptance

- A network policy has been created to allow Prometheus to scrape the `ingress-nginx-controller`.
- Prometheus targets for the `ingress-nginx-controller` should be Up, and the corresponding Grafana dashboards should be populated.

## Links

- `infra/manifests/prometheus/` and `docs/docs/prometheus-stack-install.md`
