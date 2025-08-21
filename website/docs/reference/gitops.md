---
id: gitops
slug: /reference/gitops
sidebar_position: 9
---

# GitOps Strategy

:::info This document outlines the GitOps strategy for this project, including the chosen tool and bootstrap steps. :::

## Chosen Tool: Argo CD

We have chosen Argo CD as our GitOps tool. Both Argo CD and Flux are excellent, popular open-source GitOps tools. However, we have chosen Argo CD for the following reasons:

- **User Interface:** Argo CD provides a powerful and intuitive web UI that allows for easy visualization of application state and history. This is particularly helpful for beginners and for teams that prefer a visual representation of their deployments.
- **Multi-tenancy:** Argo CD has robust multi-tenancy features, which can be beneficial as the project grows and more teams and applications are onboarded.
- **Extensibility:** Argo CD is highly extensible, with a rich ecosystem of plugins and integrations.

<details>
  <summary>Bootstrap Steps</summary>

To bootstrap Argo CD in a Kubernetes cluster, you can follow these high-level steps:

1.  **Install Argo CD:**

    ```bash
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```

2.  **Access the Argo CD UI:**

    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

    The UI will be available at `https://localhost:8080`.

3.  **Log in to Argo CD:** The initial password for the `admin` user can be retrieved with the following command:

    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```

4.  **Configure the Git repository:** Once logged in, you will need to configure Argo CD to connect to this Git repository.

5.  **Apply the application manifests:** The application manifests in the `infra/gitops/apps` directory can then be applied to the cluster: `bash kubectl apply -f infra/gitops/apps/frontend.yaml `
</details>
