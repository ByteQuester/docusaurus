---
id: environment-management
slug: /reference/environment-management
sidebar_position: 7
---

# Environment Management

:::info This document describes the strategy for managing environment-specific configurations in this project using Helm. :::

## Strategy

We use a layered approach with [Helm](https://helm.sh/) to manage configurations for different environments like `development` and `production`. This approach is based on a set of YAML files for each environment.

- **`values.yaml`**: This file, located in `charts/frontend/`, contains the default configuration values for the frontend application. These are the base values that are common across all environments.

- **`values-<env>.yaml`**: For each environment, we have a specific values file (e.g., `values-dev.yaml`, `values-prod.yaml`). These files are also located in `charts/frontend/`. They override the default values in `values.yaml` with environment-specific settings.

When deploying the application to a specific environment, we use the corresponding values file. For example, to deploy to the development environment, we would use the `values-dev.yaml` file. Our CI/CD pipeline is configured to use these files to deploy to the correct environment.

## Variable Naming Conventions

:::tip To maintain consistency, we follow these naming conventions for variables in our Helm values files:

- Variables should be named using **camelCase**.
- Variable names should be descriptive and easy to understand. :::

```yaml title="Example: charts/frontend/values.yaml"
replicaCount: 1
image:
  repository: nginx
  pullPolicy: IfNotPresent
```

## Environment-specific Configurations

Here are some examples of environment-specific configurations that we manage using this approach:

- **Replica Count**: The number of application replicas can be different for each environment. For example, we might have a single replica in `development` and multiple replicas in `production`.
- **Image Tags**: We can use different Docker image tags for each environment. For example, we might use the `dev` tag for the `development` environment and the `latest` or a specific version tag for the `production` environment.
- **Ingress Configuration**: The Ingress configuration, including hostnames and TLS settings, is different for each environment.
- **Resource Requests and Limits**: We can set different resource requests and limits for each environment to optimize resource usage.
- **Annotations**: We can add environment-specific annotations to our Kubernetes resources.
