---
id: implementing-network-policies
slug: /guides/implementing-network-policies
sidebar_position: 3
---

# Implementing Network Policies

:::info This guide provides a step-by-step process for implementing Network Policies in the EKS cluster. :::

## Prerequisites

Before you begin, you should have:

- `kubectl` installed and configured to connect to your cluster.
- A running EKS cluster with Calico installed as the Network Policy enforcement engine.

## 1. Verification

### 1.1. Verify Enforcement Engine

Confirm that a policy engine like Calico is installed and enforcing NetworkPolicy.

```bash title="Verify Calico pods"
kubectl get pods -n tigera-operator,calico-system
```

### 1.2. Namespace and Label Conventions

Ensure namespaces and application labels match policy selectors.

```bash title="Check namespaces and labels"
kubectl get ns
kubectl get deploy -A -o custom-columns=NS:.metadata.namespace,NAME:.metadata.name,LBL:.spec.selector.matchLabels
```

### 1.3. Argo Application Sanity

Verify that Argo CD applications are healthy and synced.

```bash title="Check Argo CD applications"
kubectl -n argocd get applications.argoproj.io
```

## 2. Implementation

### 2.1. Default Deny

Apply a default deny policy to the `frontend-dev` and `backend-dev` namespaces.

```bash title="Apply default deny policies"
kubectl apply -f infra/manifests/network-policies/frontend-dev/00-deny-all.yaml
kubectl apply -f infra/manifests/network-policies/backend-dev/00-deny-all.yaml
```

### 2.2. Allow DNS

Allow DNS egress from the `frontend-dev` and `backend-dev` namespaces.

```bash title="Apply allow DNS policies"
kubectl apply -f infra/manifests/network-policies/frontend-dev/allow-dns.yaml
kubectl apply -f infra/manifests/network-policies/backend-dev/allow-dns.yaml
```

### 2.3. Allow Frontend to Backend

Allow traffic from the frontend to the backend.

```bash title="Apply frontend to backend policy"
kubectl apply -f infra/manifests/network-policies/backend-dev/10-allow-from-frontend.yaml
```

### 2.4. Allow Ingress to Frontend

Allow traffic from the ingress controller to the frontend.

```bash title="Apply ingress to frontend policy"
kubectl apply -f infra/manifests/network-policies/frontend-dev/15-allow-from-ingress.yaml
```

### 2.5. Allow Ingress to Backend

Allow traffic from the ingress controller to the backend.

```bash title="Apply ingress to backend policy"
kubectl apply -f infra/manifests/network-policies/backend-dev/15-allow-from-ingress.yaml
```

### 2.6. Allow Monitoring Scrapes

Allow Prometheus to scrape metrics from the ingress controller.

```bash title="Apply monitoring scrape policy"
kubectl apply -f infra/manifests/network-policies/ingress/allow-from-prometheus.yaml
```

## 3. Validation

### 3.1. Functional Checks

Perform internal and external checks to ensure the application is functioning correctly.

```bash title="Internal check"
kubectl exec tmp-debug -n frontend-dev -- curl -sv http://backend-sample.backend-dev:80/api/health
```

```bash title="External check"
curl -sv https://dev.capitabyte.com/api/health
```

### 3.2. Apply Order and Rollback

Document the apply order of the policies and have a rollback plan.

## 4. Post-Implementation

### 4.1. Tighten Kyverno Baseline

Revisit Kyverno policies and restore intended enforcement after validating network policies.

### 4.2. Record Selectors and Enforcement Engine

Update documentation with the final namespace labels, app labels, and enforcement engine details.
