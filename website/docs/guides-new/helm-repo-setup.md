---
id: helm-repo-setup
slug: /guides/helm-repo-setup
sidebar_position: 15
---

# Helm Repository Setup

:::info This guide outlines the steps to set up and use a Helm repository with GitHub Pages. :::

## 1. Configure GitHub Pages

To host your Helm repository on GitHub Pages, you need to configure your repository settings.

1.  Navigate to your repository's **Settings** tab.
2.  In the left sidebar, click on **Pages**.
3.  Under **Source**, select the `gh-pages` branch and the `/` (root) directory as the source.
4.  Click **Save**.

:::tip Repository URL Once GitHub Pages is enabled, your Helm repository will be available at: `https://<your-github-username>.github.io/<your-repository-name>/` :::

## 2. Add the Repository Locally

To add the new Helm repository to your local machine, use the `helm repo add` command.

```bash title="Add Helm Repository"
helm repo add <repo-name> https://<your-github-username>.github.io/<your-repository-name>/
```
