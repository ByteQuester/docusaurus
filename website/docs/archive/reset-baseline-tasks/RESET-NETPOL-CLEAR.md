---
id: reset-netpol-clear
slug: /archive/reset-baseline-tasks/reset-netpol-clear
sidebar_position: 4
---

# RESET-NETPOL-CLEAR — Remove all NetworkPolicies (temporary)

- **Status**: Done

## Summary

All network policies have been deleted from the cluster to allow for baseline connectivity testing. This is a temporary measure.

For details on the execution and verification, see the [Reset Baseline Documentation](./reset-baseline.md#reset-netpol-clear--remove-all-networkpolicies-temporary).

## Steps

```bash title="Delete all NetworkPolicies"
kubectl delete netpol --all -A --kubeconfig=$HOME/.kube/config || true
```

- Ensure there are no policies left in `frontend-dev`, `backend-dev`, `ingress`, `kube-system`.

## Acceptance

- `kubectl get netpol -A` returns empty; connectivity tests can proceed unblocked.

## Rollback

- Re-apply policies from `infra/manifests/network-policies/` after baseline validation.
