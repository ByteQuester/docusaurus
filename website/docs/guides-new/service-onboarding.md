---
id: service-onboarding
slug: /guides/service-onboarding
sidebar_position: 10
---

# Service Onboarding Checklist

:::info This guide provides a step-by-step checklist for onboarding a new application or service into the platform. :::

## 1. Scaffold the Application

:::tip Start by Copying The easiest way to start is by copying an existing service that is similar to your new one from the `/apps` directory. :::

```bash title="Example for a new backend service"
cp -r apps/backend-sample apps/my-new-service
```

After copying, be sure to update the metadata in `apps/my-new-service/package.json` (like the `name` field) and remove any irrelevant code.

## 2. Create the Helm Chart

Next, create a Helm chart for your service in the `/charts` directory. We use a template to ensure consistency.

1.  **Copy the template:**

    ```bash
    cp -r charts/service-template charts/my-new-service
    ```

2.  **Update Chart Metadata:** Edit `charts/my-new-service/Chart.yaml` to set the correct `name`, `description`, and `appVersion` for your service.

## 3. Configure CI/CD

Our CI/CD pipeline has both automated and manual steps for onboarding a new service.

- **Automated Builds:** The main CI workflow (`.github/workflows/ci.yml`) automatically detects changes in `apps/*` and will run `lint`, `test`, and `build` jobs for your new service thanks to the [Smart Builds](../reference/ci-smart-build.md) logic.

:::warning Manual Step Required To publish your service's container image to our registry (GHCR), you must **manually** add a new step to the `security_and_publish` job in `.github/workflows/ci.yml`. Copy the existing `docker/build-push-action` step and update the `context` and `file` paths to point to your new service's Dockerfile. :::

## 4. Configure Helm Values

Create `values-dev.yaml` and `values-prod.yaml` inside your new chart directory (`charts/my-new-service/`). These files control the environment-specific configuration.

Start by copying the values from an existing service, then update the following key sections:

- `image.repository`: Set this to the path of your container image in GHCR.
- `ingress.hosts`: Define the hostname and path for your service.
- `ingress.annotations`: Add annotations for `cert-manager` to enable automatic TLS.

:::tip Handling CORS If your service needs to handle cross-origin requests, enable and configure the CORS settings in your `values.yaml` file. :::

```yaml title="charts/my-new-service/values-dev.yaml"
cors:
  enabled: true
  origin: '*' # Or a specific domain
  methods: 'GET,POST,PUT,DELETE,OPTIONS'
  headers: '*'
```

## 5. Deploy with GitOps

We use Argo CD to manage deployments. To deploy your new service, you need to create an Argo CD `Application` manifest.

1.  **Create the manifest:** Copy an existing manifest, for example `infra/gitops/apps/backend.yaml`, to a new file like `infra/gitops/apps/my-new-service.yaml`.

2.  **Update the manifest:** Modify the file with your service's details:
    - `metadata.name`: `my-new-service-dev`
    - `spec.destination.namespace`: The namespace to deploy to.
    - `spec.source.path`: `charts/my-new-service`
    - `spec.source.helm.valueFiles`: Point to the correct values file, e.g., `$values/charts/my-new-service/values-dev.yaml`.

Once your manifest is merged into the `main` branch, Argo CD will automatically detect and deploy your application.

## 6. Verify Deployment

After Argo CD shows the application as `Healthy` and `Synced`, verify that it's running correctly:

- Check that the pods and ingress are created in the correct namespace.
  ```bash
  kubectl get pods,ingress -n my-namespace
  ```
- Access the ingress URL and confirm you get a `200 OK` response.
- Check that the TLS certificate is valid in your browser.

:::success Onboarding Complete! Your new service is now deployed and managed by the platform. :::
