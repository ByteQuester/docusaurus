---
id: cicd-overview
slug: /reference/cicd-overview
sidebar_position: 5
---

# CI/CD Overview

:::info This document provides a high-level overview of our CI/CD pipeline, which is responsible for building, testing, and deploying our applications. :::

## Pipeline Stages

Our CI/CD pipeline consists of the following stages:

1.  **Determine Changes**: This job determines which applications have changed in the commit. This allows us to only build and test the applications that have been modified.
2.  **Build**: This job builds the container images for the changed applications.
3.  **Test**: This job runs the unit and integration tests for the changed applications.
4.  **Helm Test**: This job lints and templates the Helm charts to ensure they are valid.
5.  **Security and Publish**: This job runs security scans on the container images and publishes them to the GitHub Container Registry (GHCR).
6.  **Release**: When a new tag is pushed, this job creates a new GitHub release and attaches the container image digests to it.

:::note The full workflow is defined in the [`.github/workflows/ci.yml`](https://github.com/mpo/web-hosting/actions/workflows/ci.yml) file. :::
