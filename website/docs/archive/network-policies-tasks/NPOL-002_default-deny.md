---
id: npol-002-default-deny
slug: /archive/network-policies-tasks/npol-002-default-deny
sidebar_position: 7
---

# NPOL-002 — Default deny (ingress and egress)

- **Status**: Completed
- **Dependencies**: NPOL-001

## Scope

Apply default deny for `frontend-dev` and `backend-dev` namespaces.

## Manifests

- `infra/manifests/network-policies/frontend-dev/00-deny-all.yaml`
- `infra/manifests/network-policies/backend-dev/00-deny-all.yaml`

## Verification

First, we checked for existing network policies in the `frontend-dev` and `backend-dev` namespaces.

```bash title="Check for existing network policies"
kubectl get netpol -n frontend-dev --kubeconfig=$HOME/.kube/config
kubectl get netpol -n backend-dev --kubeconfig=$HOME/.kube/config
```

The output showed no network policies, so we applied them from the repository.

```bash title="Apply default deny policies"
kubectl apply -f infra/manifests/network-policies/frontend-dev/00-deny-all.yaml --kubeconfig=$HOME/.kube/config
kubectl apply -f infra/manifests/network-policies/backend-dev/00-deny-all.yaml --kubeconfig=$HOME/.kube/config
```

We then verified that the policies were applied successfully.

```bash title="Verify default deny policies"
kubectl get netpol -n frontend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME               POD-SELECTOR   AGE
default-deny-all   <none>         73s
```

```bash
kubectl get netpol -n backend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME               POD-SELECTOR   AGE
default-deny-all   <none>         73s
```

## Acceptance

- Both default-deny policies present; traffic blocked prior to allow rules.
