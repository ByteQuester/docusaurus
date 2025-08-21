---
id: fresh-start-redeploy
slug: /guides/fresh-start-redeploy
sidebar_position: 2
---

# Fresh Start: Sample Frontend/Backend Redeploy

:::info This guide gives you a single, practical path to redeploy the sample backend and frontend from scratch, validate connectivity, and only then reintroduce minimal policies. Follow in order. :::

:::danger Important Always pass `--kubeconfig=$HOME/.kube/config` to `kubectl`. :::

## 0) Prerequisites and Context

- Ensure `kubectl` and `helm` access works. See [Cluster Provisioning](./cluster-provisioning.md) for context.
- Keep GitOps paused for policy layers (if used). For the reset approach and rationale, see `tasks/37_minimal_connectivity_reset.md`.

## 1) Reset to a Minimal Baseline (Temporary)

- Follow the reset tasks (some may already be Done):
  - `tasks/reset-baseline/RESET-PAUSE-ARGO.md`
  - `tasks/reset-baseline/RESET-NETPOL-CLEAR.md`
  - `tasks/reset-baseline/RESET-AWS-NODE-REVERT.md`
  - `tasks/reset-baseline/RESET-RBAC-SIMPLE.md`
  - `tasks/reset-baseline/RESET-VALIDATE-CORE.md` (we’ll run again after deploy)

**Outcome**: No NetworkPolicies in effect, chart-managed ServiceAccounts, default CNI settings.

## 2) Create Namespaces

```bash title="Create Namespaces"
kubectl create ns frontend-dev --dry-run=client -o yaml | kubectl apply -f - --kubeconfig=$HOME/.kube/config
kubectl create ns backend-dev  --dry-run=client -o yaml | kubectl apply -f - --kubeconfig=$HOME/.kube/config
kubectl create ns ingress      --dry-run=client -o yaml | kubectl apply -f - --kubeconfig=$HOME/.kube/config
```

## 3) Install Ingress and Cert-Manager

- Ensure **ingress-nginx** is installed (see `tasks/00_aws_zero_to_hero.md` AWSZ-007) and configured (values in `infra/manifests/ingress-nginx/values.yaml`).
- Ensure **cert-manager** is installed (AWSZ-008) if you plan to use TLS now.

## 4) Deploy Backend Sample

```bash title="Deploy backend-sample (dev)"
helm upgrade --install backend-sample charts/backend-sample \
  -n backend-dev --create-namespace \
  --kubeconfig=$HOME/.kube/config

kubectl rollout status deploy/backend-sample -n backend-dev --kubeconfig=$HOME/.kube/config
kubectl get svc -n backend-dev --kubeconfig=$HOME/.kube/config
```

**Expected Outcome**: Service has endpoints; internal health at `/api/health` served on port 80.

## 5) Deploy Frontend Sample

:::tip Set the API base URL to reach the backend via ingress hostname or cluster DNS. For a quick internal path, you can point to the backend service name. :::

```bash title="Deploy frontend-sample (dev)"
helm upgrade --install frontend-sample charts/frontend-sample \
  -n frontend-dev --create-namespace \
  --set configMap.data.VITE_API_BASE_URL="http://backend-sample.backend-dev:80/api" \
  --kubeconfig=$HOME/.kube/config

kubectl rollout status deploy/frontend-sample -n frontend-dev --kubeconfig=$HOME/.kube/config
kubectl get svc -n frontend-dev --kubeconfig=$HOME/.kube/config
```

If using an external host `dev.<domain>`, ensure the frontend Ingress host is set in the chart values and DNS points to the ingress LB.

## 6) Core Validations

### Internal Validation

```bash title="Internal Validation (from a debug pod)"
kubectl run -n frontend-dev debug --image=nicolaka/netshoot -it --rm --restart=Never \
  --kubeconfig=$HOME/.kube/config -- \
  sh -c "curl -sv http://backend-sample.backend-dev:80/api/health && exit"
```

### External Validation

```bash title="External Validation"
curl -sv https://dev.capitabyte.com/api/health | cat || curl -sv http://dev.capitabyte.com/api/health | cat
```

:::info Troubleshooting If internal validation fails, check Service selectors and endpoints (`kubectl describe svc` and `kubectl describe deploy`). :::

## 7) Optional Steps

- **TLS**: For staging/production TLS, see `tasks/00_aws_zero_to_hero.md` AWSZ-018 / AWSZ-019.
- **Monitoring**: To set up the Prometheus stack, see the [Prometheus Stack Installation Guide](../integrations/prometheus-stack-install.md).

## 8) Reintroduce NetworkPolicies

:::warning Apply Policies with Caution Only reintroduce NetworkPolicies after all health checks are green. Apply them one by one and verify connectivity after each one. :::

- `infra/manifests/network-policies/frontend-dev/00-deny-all.yaml`
- `infra/manifests/network-policies/frontend-dev/allow-dns.yaml`
- `infra/manifests/network-policies/backend-dev/00-deny-all.yaml`
- `infra/manifests/network-policies/backend-dev/allow-dns.yaml`
- `infra/manifests/network-policies/backend-dev/10-allow-from-frontend.yaml`
- `infra/manifests/network-policies/backend-dev/15-allow-from-ingress.yaml`

## 9) Finalize Documentation

- **Portfolio Links**: Update `docs/docs/portfolio.md` and `docs/docs/integration-links.md`.
- **Verification Checklist**: Record the green checks in `docs/docs/verification-checklist.md`.
