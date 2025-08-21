---
id: backend-configuration
slug: /reference/backend-configuration
sidebar_position: 2
---

# Backend Configuration

:::info This document explains how to configure the backend application using Helm, ConfigMaps, and External Secrets. :::

## Configuration Methods

We use a combination of Helm values, ConfigMaps, and the External Secrets Operator (ESO) to configure our backend application.

<details>
  <summary>Helm Values</summary>

The primary way to configure the application is through the `values.yaml` file in the `charts/backend-sample` directory. This file allows you to set a wide range of configuration options.

```yaml title="charts/backend-sample/values.yaml"
replicaCount: 1
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: ''
```

</details>

<details>
  <summary>ConfigMap</summary>

The `ConfigMap` created by the Helm chart can be used to pass configuration to the application as environment variables. The `configMap.data` section in `values.yaml` is mounted as a `ConfigMap` and its key-value pairs can be exposed as environment variables in the Deployment.

```yaml title="charts/backend-sample/values.yaml"
configMap:
  data:
    KEY1: VALUE1
    KEY2: VALUE2
```

</details>

<details>
  <summary>Secrets</summary>

For sensitive data, we use the External Secrets Operator (ESO). This allows us to store secrets in a secure location like AWS Secrets Manager and sync them to Kubernetes Secrets.

1.  **Create a secret** in your cloud provider's secret manager.
2.  **Create an `ExternalSecret` resource** that references the secret.
3.  **ESO will sync the secret** to a Kubernetes `Secret`.
4.  **Mount the Kubernetes `Secret`** as environment variables in the Deployment.

```yaml title="charts/backend-sample/templates/externalsecret.yaml"
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-secret
spec:
  secretStoreRef:
    name: my-secret-store
    kind: ClusterSecretStore
  target:
    name: my-secret
  dataFrom:
    - extract:
        key: my-secret-key
```

</details>
