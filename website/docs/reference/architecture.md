---
id: architecture
slug: /reference/architecture
sidebar_position: 1
---

# Architecture

:::info This document provides a high-level overview of the project's architecture, including the application, infrastructure, and deployment flow. :::

## Application Architecture

The application is composed of a frontend, a backend, and an API gateway.

```mermaid
graph TD
    A[User] --> B(Frontend)
    B --> C{API Gateway}
    C --> D[Backend]
```

## Infrastructure Architecture

The infrastructure is hosted on a Kubernetes cluster, with Cloudflare for DNS and an Ingress Controller for routing traffic.

```mermaid
graph TD
    A[User] --> B(Cloudflare)
    B --> C(Ingress Controller)
    C --> D{Kubernetes Cluster}
    D --> E[Frontend Pods]
    D --> F[Backend Pods]
```

## Deployment Flow

We use a GitOps approach for deployments, with GitHub Actions for CI and Argo CD for CD.

```mermaid
graph TD
    A[Developer] --> B(Git Push)
    B --> C{GitHub Actions}
    C --> D[Build & Test]
    D --> E[Push to GHCR]
    E --> F{Argo CD}
    F --> G[Deploy to Kubernetes]
```
