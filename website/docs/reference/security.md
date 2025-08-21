---
id: security
slug: /reference/security
sidebar_position: 21
---

# Security

:::info This document provides an overview of our security strategy, including scanners, policies, secrets management, and RBAC. :::

## Scanners

We use the following scanners in our CI/CD pipeline:

- **Gitleaks**: Scans for secrets in the repository.
- **Trivy**: Scans container images for vulnerabilities.
- **Checkov**: Scans our infrastructure-as-code for misconfigurations.

## Policies

We use Kyverno to enforce admission policies in our Kubernetes cluster. These policies include:

- Requiring images to be from an allowed registry.
- Requiring resource requests and limits.
- Requiring pods to run as a non-root user.
- Verifying image signatures.

## Secrets Management

We use a combination of SOPS and the External Secrets Operator (ESO) to manage secrets.

- **SOPS**: Encrypts secrets that are stored in the Git repository.
- **ESO**: Syncs secrets from a cloud provider's secret manager to Kubernetes secrets.

## RBAC

We enforce least-privilege access for our application workloads by binding ServiceAccounts to minimal Roles.

The RBAC resources are defined in the following files:

- `infra/manifests/rbac/frontend-rbac.yaml`
- `infra/manifests/rbac/backend-rbac.yaml`

To align the ServiceAccounts used by the Helm charts with the RBAC manifests, the following values are set in the respective `values.yaml` files:

**`charts/frontend-sample/values.yaml`**

```yaml title="charts/frontend-sample/values.yaml"
serviceAccount:
  create: false
  name: 'frontend-sample'
```

**`charts/backend-sample/values.yaml`**

```yaml title="charts/backend-sample/values.yaml"
serviceAccount:
  create: false
  name: 'backend-sample'
```
