---
id: argo-reinstall
slug: /runbooks/argo-reinstall
sidebar_position: 2
---

# Reinstalling Argo CD

:::info This guide provides the steps to delete and reinstall Argo CD to resolve persistent sync issues. :::

:::danger This is a Destructive Operation Following these steps will completely remove the current Argo CD installation and all its configurations. Proceed with caution. :::

### 1. Delete Argo CD Namespaces

This will completely remove the current Argo CD installation.

```bash title="Delete Argo CD Namespaces"
kubectl delete namespace argocd
kubectl delete namespace gitops
```

### 2. Reinstall Argo CD

This will install a fresh version of Argo CD into the `argocd` namespace.

```bash title="Reinstall Argo CD"
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Re-apply Your Applications

This will deploy your applications using the manifests in your repository.

```bash title="Re-apply Applications"
kubectl apply -f infra/gitops/apps/ --recursive -n argocd
```
