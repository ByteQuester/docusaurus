---
id: argo-troubleshooting-case-study
slug: /runbooks/argo-troubleshooting-case-study
sidebar_position: 3
---

# Argo CD Troubleshooting Case Study

:::info This case study documents a real incident where Argo CD applications were not syncing correctly and were reported as Degraded. It complements the runbook in `docs/docs/argo-troubleshooting-sync-permissions.md`. :::

## Problem Statement

Argo CD applications were stuck in a `Degraded` state and were not syncing correctly. A previous attempt to reinstall Argo CD did not resolve the issue.

**Initial State:**

- Namespaces `argocd` and `gitops` existed.
- Argo CD applications were Degraded.
- A reinstall plan existed but state remained broken.

## Investigation and Resolution

Here is a step-by-step account of the troubleshooting process and the solutions applied to resolve the issue.

### 1. Reinstall Argo CD

The first step was to perform a clean reinstallation of Argo CD to ensure a known baseline.

```bash title="Reinstall Argo CD"
kubectl --kubeconfig $HOME/.kube/config delete namespace argocd || true
kubectl --kubeconfig $HOME/.kube/config delete namespace gitops || true

kubectl --kubeconfig $HOME/.kube/config create namespace argocd
kubectl --kubeconfig $HOME/.kube/config apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl --kubeconfig $HOME/.kube/config apply -f infra/gitops/apps/ --recursive -n argocd
```

### 2. Missing SSH Credentials

**Problem:** After reinstallation, Argo CD was unable to access the git repository.

:::danger Error

```
Failed to load target state: failed to generate manifest for source 1 of 1: rpc error: code = Unknown desc = failed to list refs: error creating SSH agent: "SSH agent requested but SSH_AUTH_SOCK not-specified"
```

:::

**Solution:** The SSH credentials for the git repository were missing. Applying the repository secret resolved the issue.

:::success Fix

```bash
kubectl --kubeconfig $HOME/.kube/config apply -f infra/gitops/repos/web-hosting-repo-secret.yaml
```

See also: `docs/docs/argo-quickstart.md` (deploy keys) and repo-secret hygiene in `tasks/41_gitops_repo_access.md`. :::

### 3. Missing Default Project

**Problem:** Argo CD applications were referencing a project that did not exist.

:::danger Error

```
Application referencing project default which does not exist
```

:::

**Solution:** The `default` `AppProject` was missing. Creating and applying the project manifest resolved the issue.

:::success Fix

```bash
kubectl --kubeconfig $HOME/.kube/config apply -f infra/gitops/projects/default-project.yaml
```

**Example manifest:**

```yaml title="default-project.yaml"
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: default
  namespace: argocd
spec:
  description: Default project
  sourceRepos:
    - '*'
  destinations:
    - namespace: '*'
      server: '*'
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
```

:::

### 4. HPA Metrics and Resources

**Problem:** The Horizontal Pod Autoscaler (HPA) was not functioning correctly because it could not read metrics.

**Solution:**

- **Install metrics-server:**
  ```bash
  kubectl --kubeconfig $HOME/.kube/config apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
  ```
- **Ensure Deployments set resource requests:**
  ```yaml
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
  ```

### 5. ServiceAccount for backend-sample

**Problem:** The `backend-sample` application was missing a ServiceAccount.

**Solution:** A ServiceAccount was created for the application via the Helm chart by setting `.Values.serviceAccount.create` to `true`.

### 6. Nested Application Reconciliation

**Problem:** A child application (`backend-sample`) was being managed by the `root` app-of-apps, causing a conflict.

**Solution:** The child application was deleted, allowing the `root` application to recreate and manage it correctly.

```bash
kubectl --kubeconfig $HOME/.kube/config delete application backend-sample -n argocd
```

### 7. Repo Cache Refresh

**Problem:** The Argo CD repository cache was suspected to be stale.

**Solution:** The `argocd-repo-server` deployment was restarted to force a cache refresh.

```bash
kubectl --kubeconfig $HOME/.kube/config -n argocd rollout restart deployment argocd-repo-server
```

## Root Causes

- Missing SSH credentials for Argo CD
- Missing `default` project in Argo CD
- metrics-server not installed
- Missing resource requests for HPA
- Missing ServiceAccount
- Nested Application management mismatch
- Repo cache staleness

## Outcome

:::success All issues were resolved, and the `backend-sample` application became `Synced` and `Healthy`, with the HPA functioning correctly. :::

## See Also

:::tip

- Runbook: `docs/docs/argo-troubleshooting-sync-permissions.md`
- GitOps overview: `docs/docs/gitops.md`
- App-of-apps pattern: `infra/gitops/apps/root.yaml` :::
