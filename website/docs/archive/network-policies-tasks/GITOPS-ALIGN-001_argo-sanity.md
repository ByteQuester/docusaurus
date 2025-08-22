---
id: gitops-align-001-argo-sanity
slug: /archive/network-policies-tasks/gitops-align-001-argo-sanity
sidebar_position: 3
---

# GITOPS-ALIGN-001 — Argo application sanity

- **Status**: In Progress
- **Dependencies**: AWSZ-021

## Steps

```bash title="Check Argo CD applications"
kubectl -n argocd --kubeconfig=$HOME/.kube/config get applications.argoproj.io | cat || true
kubectl -n argocd --kubeconfig=$HOME/.kube/config get app root -o yaml | cat || true
```

Verify destination server and namespaces, repo access secret, and child apps Healthy/Synced.

## Acceptance

- Argo apps show Healthy/Synced; destinations correct.
