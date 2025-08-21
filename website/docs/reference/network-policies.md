---
id: network-policies
slug: /reference/network-policies
sidebar_position: 14
---

# Network Policies

:::info This document provides an overview of the NetworkPolicies used in this project to enforce least-privilege traffic. :::

## Enforcement Engine

- **Engine:** Calico
- **Version:** v3.30.2

## Policy Files

The NetworkPolicy resources are defined in the following files:

- `infra/manifests/network-policies/backend-dev/00-deny-all.yaml`
- `infra/manifests/network-policies/backend-dev/10-allow-from-frontend.yaml`
- `infra/manifests/network-policies/backend-dev/15-allow-from-ingress.yaml`
- `infra/manifests/network-policies/backend-dev/allow-dns.yaml`
- `infra/manifests/network-policies/frontend-dev/00-deny-all.yaml`
- `infra/manifests/network-policies/frontend-dev/allow-dns.yaml`
- `infra/manifests/network-policies/frontend-dev/15-allow-from-ingress.yaml`
- `infra/manifests/network-policies/frontend-dev/allow-to-backend-dev-namespace.yaml`

## Selectors and Ports

The following selectors and ports are used in the NetworkPolicies:

### Namespaces

- `frontend-dev`: `name=frontend-dev`
- `backend-dev`: `name=backend-dev`
- `ingress`: `name=ingress`
- `kube-system`: `kubernetes.io/metadata.name=kube-system`

### Pods

- **backend-sample:** `app.kubernetes.io/name=backend-sample`
- **ingress-nginx:** `app.kubernetes.io/name=ingress-nginx`
- **kube-dns:** `k8s-app=kube-dns`

### Ports

- **Backend (from frontend):** TCP 80
- **Backend (from ingress):** TCP 3001
- **DNS:** TCP/UDP 53

:::tip Troubleshooting During the implementation of the network policies, we encountered an issue where the frontend could not communicate with the backend, resulting in a 504 Gateway Timeout error. This was due to a misconfigured egress policy on the `frontend-dev` namespace.

After extensive troubleshooting, we determined that the egress policy needed to allow traffic to the entire `backend-dev` namespace, without specifying a port or pod selector. The final working policy is defined in `infra/manifests/network-policies/frontend-dev/allow-to-backend-dev-namespace.yaml`.

This experience highlights the importance of careful policy definition and the utility of a debug pod with networking tools for troubleshooting connectivity issues. :::
