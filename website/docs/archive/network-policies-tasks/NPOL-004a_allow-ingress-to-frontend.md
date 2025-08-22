---
id: npol-004a-allow-ingress-to-frontend
slug: /archive/network-policies-tasks/npol-004a-allow-ingress-to-frontend
sidebar_position: 10
---

# NPOL-004a — Allow ingress-controller → frontend traffic

- **Status**: Completed
- **Dependencies**: NPOL-002

## Scope

This task was created to address a 504 Gateway Timeout error on the frontend, which was caused by a missing network policy to allow traffic from the ingress controller to the frontend pods.

## Manifests

- `infra/manifests/network-policies/frontend-dev/15-allow-from-ingress.yaml`

## Verification

We first identified that the `default-deny-all` policy in the `frontend-dev` namespace was blocking traffic from the ingress controller. We then created and applied a new network policy to allow this traffic.

```bash title="Apply allow-from-ingress policy"
kubectl apply -f infra/manifests/network-policies/frontend-dev/15-allow-from-ingress.yaml --kubeconfig=$HOME/.kube/config
```

We then verified that the policy was applied successfully.

```bash title="Verify allow-from-ingress policy"
kubectl get netpol -n frontend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME                   POD-SELECTOR                            AGE
allow-dns              <none>                                  10m
allow-from-ingress     app.kubernetes.io/name=frontend-sample  1m
default-deny-all       <none>                                  15m
```

## Acceptance

- The 504 Gateway Timeout error on the frontend should be resolved.
