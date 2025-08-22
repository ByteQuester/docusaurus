---
id: argo-reinstall
slug: /runbooks/argo-reinstall
sidebar_position: 2
---

# Reinstalling Argo CD

:::info This guide provides the steps to delete and reinstall Argo CD. This is typically done as a last resort to resolve persistent sync issues or other critical errors that cannot be resolved through normal troubleshooting. :::

:::danger This is a Destructive Operation Following these steps will completely remove the current Argo CD installation and all its configurations, including projects, applications, and settings. Proceed with caution and ensure you have backups or a way to restore your application definitions. :::

## Prerequisites

Before you begin, you should have:

- `kubectl` installed and configured to connect to your cluster.
- Access to your Kubernetes manifests repository.

## 1. Delete Argo CD Namespaces

This step will completely remove the current Argo CD installation and all its associated resources from the cluster.

```bash title="Delete Argo CD Namespaces"
kubectl delete namespace argocd
kubectl delete namespace gitops
```

## 2. Reinstall Argo CD

This will install a fresh, stable version of Argo CD into a new `argocd` namespace.

```bash title="Reinstall Argo CD"
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 3. Re-apply Your Applications

Once Argo CD is reinstalled, you need to re-apply your application definitions. This will redeploy your applications using the manifests from your repository.

```bash title="Re-apply Applications"
kubectl apply -f infra/gitops/apps/ --recursive -n argocd
```

## 4. Verification

After re-applying your applications, you should verify that everything is working as expected.

1.  **Check the Argo CD UI:** Access the Argo CD UI and ensure that your applications are listed and are in a `Healthy` and `Synced` state.
2.  **Verify Application Endpoints:** Check the endpoints of your applications to ensure they are accessible and functioning correctly.
