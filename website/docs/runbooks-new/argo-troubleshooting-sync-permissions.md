---
id: argo-troubleshooting-sync-permissions
slug: /runbooks/argo-troubleshooting-sync-permissions
sidebar_position: 4
---

# Argo CD: Troubleshooting PermissionDenied and Non-Watched Namespaces

:::info This playbook documents how to diagnose and fix Argo CD `PermissionDenied` errors and cases where the controller ignores Application resources because they live outside the watched namespace. :::

## Scope and Symptoms

- You see `PermissionDenied` when syncing via Argo CLI, or apps stay `OutOfSync` without controller action.
- Argo CD only processes Application objects in `argocd`, but your apps were created in another namespace (e.g., `gitops`).

## Root Cause

:::note By default, Argo CD watches Application resources in its own control-plane namespace (`argocd`). If Application resources are created in another namespace, Argo CD will ignore them unless all of the following are configured:

- `argocd-cm` includes `application.namespaces: <other-namespace>`
- The relevant `AppProject` has `spec.sourceNamespaces` including that other namespace.
- Appropriate RBAC allows the operation. :::

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem';

<Tabs>
  <TabItem value="fast-fix" label="Fast Fix (Recommended)">

Use this when you don’t need multi-namespace Application definitions.

1.  **Ensure Application manifests target `argocd`:**

    ```yaml title="Application manifest snippet"
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: <app-name>
      namespace: argocd
    spec:
      destination:
        server: https://kubernetes.default.svc
        # The app’s workload (Helm/K8s) destination namespace can be anything,
        # but the Application resource itself lives in argocd
    ```

    For the root app-of-apps, ensure:

    ```yaml
    spec:
      destination:
        server: https://kubernetes.default.svc
        namespace: argocd
    ```

2.  **Remove any Applications created in other namespaces (e.g., `gitops`):**

    ```bash
    kubectl -n gitops delete application root frontend-sample backend-sample --ignore-not-found
    ```

3.  **Apply the corrected files:**

    ```bash
    kubectl apply -f infra/gitops/apps/root.yaml
    kubectl apply -f infra/gitops/apps/frontend-sample.yaml
    kubectl apply -f infra/gitops/apps/backend-sample.yaml
    ```

4.  **Trigger a hard refresh and verify:**

    ```bash
    kubectl -n argocd annotate application root argocd.argoproj.io/refresh=hard --overwrite
    kubectl -n argocd annotate application frontend-sample argocd.argoproj.io/refresh=hard --overwrite
    kubectl -n argocd annotate application backend-sample argocd.argoproj.io/refresh=hard --overwrite

    kubectl -n argocd get application frontend-sample -o yaml | yq ".status.sync.status,.status.health.status"
    kubectl -n argocd get application backend-sample -o yaml | yq ".status.sync.status,.status.health.status"
    ```

    </TabItem>
    <TabItem value="multi-namespace" label="Alternative: Multi-Namespace Management">

Use this when you intentionally want Application CRs to live outside `argocd` (e.g., `gitops`).

1.  **Allow Argo to watch the namespace:**

    ```bash
    kubectl -n argocd patch cm argocd-cm --type merge -p '{"data":{"application.namespaces":"gitops"}}'
    ```

2.  **Restart components to pick up changes:**

    ```bash
    kubectl -n argocd rollout restart deployment/argocd-server
    kubectl -n argocd rollout restart deployment/argocd-repo-server || true
    kubectl -n argocd rollout restart statefulset/argocd-application-controller
    ```

3.  **Update project to accept sources from `gitops`:**

    ```yaml title="project-default-source-ns.yaml"
    apiVersion: argoproj.io/v1alpha1
    kind: AppProject
    metadata:
      name: default
      namespace: argocd
    spec:
      sourceRepos:
        - '*'
      destinations:
        - namespace: '*'
          server: https://kubernetes.default.svc
      sourceNamespaces:
        - argocd
        - gitops
    ```

    Apply once:

    ```bash
    kubectl apply -f project-default-source-ns.yaml
    ```

4.  Keep Applications in `gitops` and re-annotate to refresh as in the Fast fix. </TabItem> </Tabs>

## Quick Triage

:::tip If you are still blocked, here are some quick triage steps:

- **Inspect Application conditions:**
  ```bash
  kubectl -n argocd get app frontend-sample -o jsonpath='{.status.conditions[*].message}' ; echo
  ```
- **Controller logs:**
  ```bash
  kubectl -n argocd logs sts/argocd-application-controller -c argocd-application-controller | tail -n 200
  ```
- **Repo access secret present and correct:**
  ```bash
  kubectl -n argocd get secret -l argocd.argoproj.io/secret-type=repository
  ```
- **Destination:** - Expect `spec.destination.server: https://kubernetes.default.svc` in all Applications :::

## Security Note: Repository Secrets

:::danger Do not commit private keys to git. Rotate credentials if a key was committed and recreate the repo secret directly in the cluster:

```bash
kubectl -n argocd delete secret web-hosting-repo-secret || true
kubectl -n argocd create secret generic web-hosting-repo-secret \
  --from-literal=url=git@github.com:ByteQuester/hosting-monorepo.git \
  --from-file=sshPrivateKey=/secure/path/to/id_ed25519
kubectl -n argocd label secret web-hosting-repo-secret argocd.argoproj.io/secret-type=repository --overwrite
```

:::

## Related docs

- [GitOps overview](../gitops.md)
- [ArgoCD quickstart and deploy keys](./argo-quickstart.md)
- [Service onboarding flow](../service-onboarding.md)
- [Case Study](./argo-troubleshooting-case-study.md)
