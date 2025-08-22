---
id: rbac-inv-001-inventory
slug: /archive/rbac-tasks/rbac-inv-001-inventory
sidebar_position: 4
---

# RBAC-INV-001 — Inventory current RBAC

- **Status**: Completed
- **Dependencies**: AWSZ-006

## Commands

```bash title="Inventory RBAC resources"
kubectl get sa,role,rolebinding -n frontend-dev --kubeconfig=$HOME/.kube/config
kubectl get sa,role,rolebinding -n backend-dev --kubeconfig=$HOME/.kube/config
```

## Findings

- **frontend-dev**:
  - ServiceAccounts: default, frontend, frontend-sample
  - Roles: none
  - RoleBindings: none
  - Deployment `frontend-sample` uses ServiceAccount `frontend-sample` (chart-managed)
- **backend-dev**:
  - ServiceAccounts: default, backend-sample
  - Roles: none
  - RoleBindings: none
  - Deployment `backend-sample` uses ServiceAccount `backend-sample` (chart-managed)

## Acceptance

- Inventory recorded; naming sources (chart vs. manifest) identified.
