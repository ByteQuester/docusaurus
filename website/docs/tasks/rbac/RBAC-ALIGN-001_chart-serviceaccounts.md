# RBAC-ALIGN-001 — Align chart ServiceAccounts with cluster SAs

- Status: In Progress
- Dependencies: RBAC-INV-001

## Context

Charts currently create their own ServiceAccounts by default:

- `charts/frontend-sample/values.yaml`: `serviceAccount.create: true`, `name: ""` → SA name resolves to chart fullname (likely `frontend-sample`).
- `charts/backend-sample/values.yaml`: `serviceAccount.create: true`, `name: ""` → SA name resolves to chart fullname (likely `backend-sample`).

Standalone manifests define SAs:

- `infra/manifests/rbac/frontend-rbac.yaml` defines `ServiceAccount frontend` in `frontend-dev`.
- `infra/manifests/rbac/backend-rbac.yaml` defines `ServiceAccount backend-sample` in `backend-dev`.

## Decision

Adopt “manifests as source of truth” to keep RBAC central:

- Set charts to reuse precreated ServiceAccounts:
  - Update values for both charts: `serviceAccount.create: false`, `serviceAccount.name: <precreated-sa>`.
- Names to use:
  - frontend-sample chart → `serviceAccount.name: frontend` (matches manifest)
  - backend-sample chart → `serviceAccount.name: backend-sample` (matches manifest)

## Steps

1. Update Helm values:

```yaml
# charts/frontend-sample/values.yaml
serviceAccount:
  create: false
  name: frontend

# charts/backend-sample/values.yaml
serviceAccount:
  create: false
  name: backend-sample
```

2. Apply RBAC manifests (if not already):

```bash
kubectl apply -f infra/manifests/rbac/frontend-rbac.yaml --kubeconfig=$HOME/.kube/config
kubectl apply -f infra/manifests/rbac/backend-rbac.yaml --kubeconfig=$HOME/.kube/config
```

3. Redeploy charts (Argo sync or helm upgrade) and verify pods use intended ServiceAccounts.

## Verification

```bash
kubectl get pod -n frontend-dev -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.serviceAccountName}{"\n"}{end}' --kubeconfig=$HOME/.kube/config | cat
kubectl get pod -n backend-dev -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.serviceAccountName}{"\n"}{end}' --kubeconfig=$HOME/.kube/config | cat
```

## Acceptance

- Charts stopped creating SAs; pods use precreated ServiceAccounts defined in manifests.
