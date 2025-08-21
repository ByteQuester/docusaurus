---
id: argo-quickstart
slug: /runbooks/argo-quickstart
sidebar_position: 1
---

# ArgoCD Quickstart

:::info This document provides a quick start guide for setting up ArgoCD. :::

## Repository Access

:::note Deploy Keys To allow ArgoCD to access the git repository, we use a deploy key. A deploy key is an SSH key that grants access to a single repository. :::

### 1. Generating a Deploy Key

Generate a new SSH keypair:

```bash title="Generate SSH key"
ssh-keygen -t ed25519 -C "argocd-deploy-key" -f ./argocd-deploy-key
```

### 2. Adding the Deploy Key to GitHub

1.  Go to your repository's **Settings** on GitHub.
2.  Click on **Deploy Keys**.
3.  Click **Add deploy key**.
4.  Give it a title, e.g., "ArgoCD".
5.  Paste the public key (`argocd-deploy-key.pub`) into the "Key" field.
6.  Do not check "Allow write access".
7.  Click **Add key**.
