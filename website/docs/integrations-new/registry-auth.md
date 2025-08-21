---
id: registry-auth-and-images
slug: /integrations/registry-auth-and-images
sidebar_position: 22
---

# Registry Auth & Image References

:::info This guide explains how to handle registry authentication and image references in your Kubernetes deployments. :::

## Image Naming and Verification

- **CI Image Naming**: The default image name is `ghcr.io/<owner>/<repo>`. Note that the owner is lowercase in GHCR (e.g., `ghcr.io/bytequester/hosting-monorepo`).
- **Verify Tags**: - You can see the available tags in the **Packages** section of your GitHub repository. - Alternatively, you can use the `docker manifest inspect` command: ```bash docker manifest inspect ghcr.io/<owner>/<repo>:<tag>

````

## Creating a Namespace Pull Secret

To allow your cluster to pull images from a private registry, you need to create a `docker-registry` secret in each namespace that needs it.

```bash title="Create ghcr-credentials secret"
kubectl create secret docker-registry ghcr-credentials \
  -n <namespace> --docker-server=ghcr.io \
  --docker-username=<user> --docker-password=<PAT> \
  --dry-run=client -o yaml | kubectl apply -f -
````

:::tip

- Wire this secret to your workloads via Helm (`.Values.imagePullSecrets`) or by patching the ServiceAccount.
- If access fails, ensure your Personal Access Token (PAT) has the `read:packages` scope, has been approved for SSO if required, or make the package public. :::

## Pinning by SHA Tags

:::note Immutable Tags It is best practice to use immutable tags (like the Git SHA) to avoid unexpected changes to your deployments. :::

If your CI/CD pipeline publishes images with SHA tags, you should pin your deployments to the exact SHA.

**Example:** `c6cd88ef7de43520f6e95fcb87e6f2452b45edd4`

### Steps to Verify and Use a SHA Tag:

1.  **Verify the manifest:**
    ```bash
    docker manifest inspect ghcr.io/bytequester/hosting-monorepo:c6cd88ef7de43520f6e95fcb87e6f2452b45edd4 | jq .schemaVersion
    ```
2.  **Update your Helm chart values:** In `charts/frontend/values-dev.yaml`, set the `image.repository` and `image.tag`:
    ```yaml
    image:
      repository: ghcr.io/bytequester/hosting-monorepo
      tag: c6cd88ef7de43520f6e95fcb87e6f2452b45edd4
    ```
3.  **Redeploy and verify:** Redeploy your application and ensure the pods become `Ready`.

:::tip Short SHA Tags Some CI setups publish short SHA tags like `sha-<short>`. If so, use `crane ls` or the GitHub Packages UI to copy the exact tag string. :::
