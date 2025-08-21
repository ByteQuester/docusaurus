---
id: gitops-app-of-apps
slug: /reference/gitops-app-of-apps
sidebar_position: 8
---

# App-of-Apps GitOps Pattern

:::info This document explains the app-of-apps pattern used in this repository for managing applications with Argo CD. :::

## Overview

The app-of-apps pattern is a powerful way to manage a large number of applications in a declarative and scalable way. It involves using a single "root" Argo CD application to manage other "child" applications.

In this repository, the `root` application is defined in `infra/gitops/apps/root.yaml`. This application is responsible for syncing all the other applications defined in the `infra/gitops/apps` directory.

## Root Application

The `root.yaml` file looks like this:

```yaml title="infra/gitops/apps/root.yaml"
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'git@github.com:ByteQuester/hosting-monorepo.git' # Replace with your repository URL
    path: infra/gitops/apps
    targetRevision: HEAD
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

This application syncs all the YAML files in the `infra/gitops/apps` directory. This means that any new application added to this directory will be automatically synced by the `root` application.

## Child Application

A child application is a regular Argo CD application that is managed by the `root` application. The manifest for a child application defines how the application should be deployed.

Here is an example of the `backend-sample` child application, located at `infra/gitops/apps/backend-sample.yaml`:

```yaml title="infra/gitops/apps/backend-sample.yaml"
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: backend-sample
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'git@github.com:ByteQuester/hosting-monorepo.git' # Replace with your repository URL
    path: charts/backend-sample
    targetRevision: HEAD
    helm:
      valueFiles:
        - values.yaml
      values: |
        ingress:
          hosts:
            - host: dev.capitabyte.com
              paths:
                - path: /api
                  pathType: ImplementationSpecific
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: backend-dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Helm Chart Structure

Each application has a corresponding Helm chart in the `charts` directory. The `Chart.yaml` file defines the metadata for the chart.

Here is an example of the `Chart.yaml` for the `backend-sample` application, located at `charts/backend-sample/Chart.yaml`:

```yaml title="charts/backend-sample/Chart.yaml"
apiVersion: v2
name: backend-sample
description: A Helm chart for the backend sample application
version: 0.1.0
appVersion: '1.0'
```

<details>
  <summary>Adding a New Service Application</summary>

To add a new service application to the app-of-apps structure, you need to create a new Argo CD application manifest in the `infra/gitops/apps` directory.

Here is a step-by-step guide:

1.  **Create a new Helm chart for your application:**

    Create a new Helm chart for your application in the `charts` directory. You can use the `service-template` as a starting point. For more details on creating Helm charts, see the Helm Charts Overview.

2.  **Create a new Argo CD application manifest:**

    Create a new YAML file in the `infra/gitops/apps` directory. This file will define the Argo CD application for your new service. You can use the `backend-sample` or `frontend-sample` applications as examples.

3.  **Commit and push the changes:**

    Commit the new Helm chart and the new application manifest to the git repository and push the changes.

Argo CD will automatically detect the new application and sync it to the cluster. You can check the status of the new application in the Argo CD UI.

For a more detailed guide on onboarding a new service, see the [Service Onboarding Cheat Sheet](../guides/service-onboarding.md).

</details>

:::note **Related Documents**

- [GitOps Overview](gitops.md)
- [Argo CD Quickstart](../runbooks/argo-quickstart.md)
- [Service Onboarding Cheat Sheet](../guides/service-onboarding.md) :::
