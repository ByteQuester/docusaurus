---
id: argo-quickstart
slug: /runbooks/argo-quickstart
sidebar_position: 1
---

# ArgoCD Quickstart

:::info This document provides a quick start guide for setting up ArgoCD, a declarative, GitOps continuous delivery tool for Kubernetes. :::

## Prerequisites

Before you begin, you should have:

- A running Kubernetes cluster.
- `kubectl` installed and configured to connect to your cluster.
- A GitHub repository containing your Kubernetes manifests.

## 1. Deploy Key Setup

To allow ArgoCD to access your git repository, we use a deploy key. A deploy key is an SSH key that grants read-only access to a single repository.

### 1.1. Generating a Deploy Key

First, generate a new SSH keypair. This key will be used to authenticate ArgoCD with your GitHub repository.

```bash title="Generate SSH key"
ssh-keygen -t ed25519 -C "argocd-deploy-key" -f ./argocd-deploy-key
```

This command will create two files: `argocd-deploy-key` (the private key) and `argocd-deploy-key.pub` (the public key).

### 1.2. Adding the Deploy Key to GitHub

Next, you need to add the public key as a deploy key to your GitHub repository.

1.  Navigate to your repository's **Settings** page on GitHub.
2.  In the sidebar, click on **Deploy Keys**.
3.  Click the **Add deploy key** button.
4.  Provide a **Title** for the key, for example, "ArgoCD".
5.  Paste the contents of the public key file (`argocd-deploy-key.pub`) into the **Key** field.
6.  Ensure the **Allow write access** checkbox is **not** checked, as ArgoCD only needs read access.
7.  Click **Add key** to save the deploy key.
