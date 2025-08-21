---
id: secrets-management
slug: /reference/secrets-management
sidebar_position: 20
---

# Secrets Management Strategy

:::info This document outlines the strategy for managing secrets in this project, covering both secrets stored in Git and secrets managed in a cloud provider. :::

## 1. Encrypted Secrets in Git: SOPS

For secrets that need to be stored in the Git repository (e.g., Helm values with sensitive data), we will use [SOPS (Secrets OPerationS)](https://github.com/getsops/sops) with [age](https://github.com/FiloSottile/age) for encryption.

### Key Management

- **age keys**: We will use `age` for simple and secure key management. A master key pair will be generated and the private key will be securely shared with authorized developers and the CI/CD system.
- **Encryption**: Before committing any file containing secrets, it must be encrypted with SOPS and the public age key.

### Workflow

1.  **Create a `.sops.yaml`** file in the root of the repository to define the encryption rules.
2.  **Encrypt files**: `sops --encrypt --in-place my-secrets.yaml`
3.  **Decrypt files**: `sops --decrypt --in-place my-secrets.yaml`

CI/CD pipelines will be configured with the private age key to decrypt secrets during deployment.

## 2. Runtime Secrets: External Secrets Operator (ESO)

For secrets that are managed by a cloud provider's secret manager (e.g., AWS Secrets Manager, Google Secret Manager), we will use the [External Secrets Operator (ESO)](https://external-secrets.io/).

### How it Works

1.  **Store secrets** in your cloud provider's secret manager.
2.  **Create an `ExternalSecret`** Kubernetes resource that references the secret in the external provider.
3.  **ESO syncs** the secret from the external provider and creates a native Kubernetes `Secret`.
4.  **Pods mount** the Kubernetes `Secret`.

:::tip This approach keeps secrets out of the Git repository and leverages the security and auditing features of the cloud provider's secret manager. :::

## 3. Secret Lifecycle

- **Creation**: Secrets are created either by encrypting them with SOPS and committing them to Git, or by adding them to an external secret manager.
- **Rotation**: Secrets should be rotated regularly. For secrets managed by ESO, this can often be automated in the cloud provider. For SOPS-encrypted secrets, a manual process of updating and re-encrypting the file is required.
- **Deletion**: When a secret is no longer needed, it should be removed from the secret manager or the Git repository.
