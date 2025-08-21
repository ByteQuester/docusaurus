# RBAC-APPLY-001 — Apply RBAC manifests

- Status: In Progress
- Dependencies: RBAC-ALIGN-001

## Steps

```bash
kubectl apply -f infra/manifests/rbac/frontend-rbac.yaml --kubeconfig=$HOME/.kube/config
kubectl apply -f infra/manifests/rbac/backend-rbac.yaml --kubeconfig=$HOME/.kube/config
```

## Acceptance

- ServiceAccounts, Roles, and RoleBindings exist in `frontend-dev` and `backend-dev` with intended names.
