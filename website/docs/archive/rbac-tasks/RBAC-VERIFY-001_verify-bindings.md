---
id: rbac-verify-001-verify-bindings
slug: /archive/rbac-tasks/rbac-verify-001-verify-bindings
sidebar_position: 7
---

# RBAC-VERIFY-001 — Verify RoleBindings and pod ServiceAccounts

- **Status**: Planned
- **Dependencies**: RBAC-APPLY-001

## Commands

```bash title="Verify RoleBindings and ServiceAccounts"
kubectl get role,rolebinding -n frontend-dev --kubeconfig=$HOME/.kube/config
kubectl get role,rolebinding -n backend-dev --kubeconfig=$HOME/.kube/config
kubectl get pod -n frontend-dev -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.serviceAccountName}{"\n"}{end}' --kubeconfig=$HOME/.kube/config | cat
kubectl get pod -n backend-dev -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.serviceAccountName}{"\n"}{end}' --kubeconfig=$HOME/.kube/config | cat
```

## Acceptance

- RoleBindings target the intended ServiceAccounts; pods run with those SAs.
