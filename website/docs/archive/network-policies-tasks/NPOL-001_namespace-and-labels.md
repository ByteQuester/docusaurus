---
id: npol-001-namespace-and-labels
slug: /archive/network-policies-tasks/npol-001-namespace-and-labels
sidebar_position: 6
---

# NPOL-001 — Namespace and label conventions

- **Status**: In Progress
- **Dependencies**: AWSZ-015, AWSZ-016

## Scope

Ensure namespaces and application labels match policy selectors.

## Steps

```bash title="Check namespaces and labels"
kubectl get ns --kubeconfig=$HOME/.kube/config | egrep "frontend-dev|backend-dev|ingress|monitoring|kube-system" | cat
kubectl get deploy -A --kubeconfig=$HOME/.kube/config -o custom-columns=NS:.metadata.namespace,NAME:.metadata.name,LBL:.spec.selector.matchLabels | cat
```

Align labels to:

- Namespaces: `name=frontend-dev|backend-dev|ingress|monitoring`
- Apps: `app=frontend|backend`

## Acceptance

- Namespaces and labels exist as above and are documented in `docs/docs/observability.md`.

## Links

- `docs/docs/observability.md` — Network Policy Selectors
