# RBAC-ROLLBACK-001 — Rollback plan

- Status: Planned
- Dependencies: RBAC-APPLY-001

## Commands

```bash
kubectl delete -f infra/manifests/rbac/frontend-rbac.yaml --kubeconfig=$HOME/.kube/config || true
kubectl delete -f infra/manifests/rbac/backend-rbac.yaml --kubeconfig=$HOME/.kube/config || true
```

## Acceptance

- RBAC resources removed; apps still run (they should not require Kubernetes API access by default).
