---
id: implementing-rbac
slug: /guides/implementing-rbac
sidebar_position: 4
---

# Implementing RBAC

:::info This guide provides a step-by-step process for implementing Role-Based Access Control (RBAC) in the EKS cluster. :::

## 1. Inventory

### 1.1. Inventory current RBAC setup

Before making any changes, it is important to understand the current RBAC configuration.

```bash title="Inventory RBAC resources"
kubectl get sa,role,rolebinding -n frontend-dev
kubectl get sa,role,rolebinding -n backend-dev
```

## 2. Alignment

### 2.1. Align chart ServiceAccounts with cluster SAs

Adopt “manifests as source of truth” to keep RBAC central.

- **Set charts to reuse precreated ServiceAccounts:**
  - Update values for both charts: `serviceAccount.create: false`, `serviceAccount.name: <precreated-sa>`.
- **Names to use:**
  - frontend-sample chart → `serviceAccount.name: frontend` (matches manifest)
  - backend-sample chart → `serviceAccount.name: backend-sample` (matches manifest)

## 3. Application

### 3.1. Apply RBAC manifests

Apply the RBAC manifests to create the necessary ServiceAccounts, Roles, and RoleBindings.

```bash title="Apply RBAC manifests"
kubectl apply -f infra/manifests/rbac/frontend-rbac.yaml
kubectl apply -f infra/manifests/rbac/backend-rbac.yaml
```

## 4. Verification

### 4.1. Verify RoleBindings and pod ServiceAccounts

Verify that the RoleBindings target the intended ServiceAccounts and that the pods are running with those SAs.

```bash title="Verify RoleBindings and ServiceAccounts"
kubectl get role,rolebinding -n frontend-dev
kubectl get role,rolebinding -n backend-dev
kubectl get pod -n frontend-dev -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.serviceAccountName}{"\n"}{end}'
kubectl get pod -n backend-dev -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.serviceAccountName}{"\n"}{end}'
```

### 4.2. Runtime behavior checks

Tail the logs of the applications to ensure no RBAC Forbidden errors appear.

```bash title="Check application logs for RBAC errors"
kubectl logs -n frontend-dev deploy/frontend-sample | egrep -i "forbidden|rbac|permission|denied" || true
kubectl logs -n backend-dev deploy/backend-sample | egrep -i "forbidden|rbac|permission|denied" || true
```

## 5. Rollback

### 5.1. Rollback plan

If something goes wrong, you can roll back the changes by deleting the RBAC resources.

```bash title="Rollback RBAC changes"
kubectl delete -f infra/manifests/rbac/frontend-rbac.yaml || true
kubectl delete -f infra/manifests/rbac/backend-rbac.yaml || true
```

## 6. Documentation

### 6.1. Update security docs

Update the security documentation with the chosen alignment approach and links to the RBAC manifests.

```

```
