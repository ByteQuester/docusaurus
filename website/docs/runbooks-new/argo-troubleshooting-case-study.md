---
id: argo-troubleshooting-case-study
slug: /runbooks/argo-troubleshooting-case-study
sidebar_position: 3
---

# Argo CD Troubleshooting Case Study

:::info This case study documents a real incident where Argo CD applications were not syncing correctly and were reported as Degraded. It complements the runbook in `docs/docs/argo-troubleshooting-sync-permissions.md`. :::

## Initial State

- Namespaces `argocd` and `gitops` existed
- Argo CD applications were Degraded
- A reinstall plan existed but state remained broken

## Steps Taken

### 1. Reinstall Argo CD

```bash title="Reinstall Argo CD"
kubectl --kubeconfig $HOME/.kube/config delete namespace argocd || true
kubectl --kubeconfig $HOME/.kube/config delete namespace gitops || true

kubectl --kubeconfig $HOME/.kube/config create namespace argocd
kubectl --kubeconfig $HOME/.kube/config apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl --kubeconfig $HOME/.kube/config apply -f infra/gitops/apps/ --recursive -n argocd
```

### 2. Missing SSH credentials

:::danger Error

```
Failed to load target state: failed to generate manifest for source 1 of 1: rpc error: code = Unknown desc = failed to list refs: error creating SSH agent: "SSH agent requested but SSH_AUTH_SOCK not-specified"
```

:::

:::success Fix Apply the repository secret (create out of band; do not commit real keys):

```bash
kubectl --kubeconfig $HOME/.kube/config apply -f infra/gitops/repos/web-hosting-repo-secret.yaml
```

See also: `docs/docs/argo-quickstart.md` (deploy keys) and repo-secret hygiene in `tasks/41_gitops_repo_access.md`. :::

### 3. Missing default project

:::danger Error

```
Application referencing project default which does not exist
```

:::

:::success Fix Create the `default` `AppProject` and apply it:

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

### 4. HPA metrics and resources

- **Install metrics-server:**
  ```bash
  kubectl --kubeconfig $HOME/.kube/config apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
  ```
- **Ensure Deployments set resource requests so HPA can read metrics, e.g.:**
  ```yaml
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
  ```

### 5. ServiceAccount for backend-sample

Create a ServiceAccount via Helm template when `.Values.serviceAccount.create` is true (see chart template snippet in repo).

### 6. Nested application reconciliation

If a child app (e.g., `backend-sample`) is managed by the `root` app-of-apps, delete the child and let `root` recreate it:

```bash
kubectl --kubeconfig $HOME/.kube/config delete application backend-sample -n argocd
```

### 7. Repo cache refresh

Restart repo-server when cache staleness is suspected:

```bash
kubectl --kubeconfig $HOME/.kube/config -n argocd rollout restart deployment argocd-repo-server
```

## Root Causes

:::note

- Missing SSH credentials for Argo CD
- Missing `default` project in Argo CD
- metrics-server not installed
- Missing resource requests for HPA
- Missing ServiceAccount
- Nested Application management mismatch
- Repo cache staleness :::

## Outcome

:::success All issues resolved; `backend-sample` became Synced and Healthy, HPA functioning. :::

## See also

:::tip

- Runbook: `docs/docs/argo-troubleshooting-sync-permissions.md`
- GitOps overview: `docs/docs/gitops.md`
- App-of-apps pattern: `infra/gitops/apps/root.yaml` :::
