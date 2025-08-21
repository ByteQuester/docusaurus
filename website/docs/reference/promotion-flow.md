---
id: promotion-flow
slug: /reference/promotion-flow
sidebar_position: 17
---

# Promotion Flow: Dev to Prod

:::info This document outlines the process for promoting application changes from the development environment to the production environment. :::

## Environment-specific Values

Our Helm charts are configured to use environment-specific values files:

- `values-dev.yaml`: for the development environment.
- `values-prod.yaml`: for the production environment.

These files are located in the respective chart directories (e.g., `charts/frontend/`).

The Argo CD Application manifests are configured to use the appropriate values file for each environment.

<details>
  <summary>Promotion Process</summary>

Promoting a new version of an application to production is done by updating the image tag in the `values-prod.yaml` file.

1.  **Create a new branch**:

    ```bash
    git checkout -b feat/promote-frontend-v1.2.3
    ```

2.  **Update the image tag**: In the `charts/frontend/values-prod.yaml` file, update the `image.tag` value to the new version.

    ```yaml title="charts/frontend/values-prod.yaml"
    # charts/frontend/values-prod.yaml
    image:
      tag: v1.2.3
    ```

3.  **Commit and push the change**:

    ```bash
    git add charts/frontend/values-prod.yaml
    git commit -m "Promote frontend to v1.2.3"
    git push origin feat/promote-frontend-v1.2.3
    ```

4.  **Create a Pull Request**: Open a pull request from your feature branch to the `main` branch.

5.  **Review and Merge**: The pull request should be reviewed by another team member. Once approved, it can be merged.
</details>

## Automatic Deployment

Once the pull request is merged to `main`, Argo CD will detect the change in the Git repository and automatically sync the application, deploying the new version to the production environment.
