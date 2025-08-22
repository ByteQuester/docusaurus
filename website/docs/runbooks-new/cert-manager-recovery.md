---
id: cert-manager-recovery
slug: /runbooks/cert-manager-recovery
sidebar_position: 5
---

# Cert-Manager Recovery

:::info This runbook provides a set of atomic tasks to resolve inconsistent Helm/CRD state for cert-manager and complete the TLS setup. :::

:::danger Important Only proceed if you have cluster-admin access. If this is a shared cluster with existing cert-manager consumers, coordinate with your team first. :::

## Prerequisites

Before you begin, you should have:

- `kubectl` and `helm` installed and configured to connect to your cluster.
- Cluster-admin access to the Kubernetes cluster.

## 1. Cleanup Existing Installation

### 1.1. Align kube context

Ensure `helm` and `kubectl` point to the same cluster/context and kubeconfig.

```bash title="Check kubectl and helm contexts"
kubectl config current-context
helm ls -A
```

### 1.2. Snapshot current state

Collect state for later reference and rollback if needed.

```bash title="Snapshot cert-manager state"
helm list -A | grep cert-manager
kubectl get ns cert-manager -o yaml
kubectl get all -n cert-manager
kubectl get crd | grep cert-manager.io
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations | grep cert-manager
```

### 1.3. Uninstall existing release

If `helm ls` shows a release, run:

```bash title="Uninstall cert-manager"
helm uninstall cert-manager -n cert-manager || true
```

### 1.4. Purge Helm release secrets

```bash title="Delete cert-manager secrets"
kubectl -n cert-manager delete secret -l owner=helm,name=cert-manager || true
```

### 1.5. Remove stale CRDs (only if safe)

**Preconditions:** Confirm no in-cluster resources rely on cert-manager (Certificates, Issuers). If this is a fresh setup, proceed.

```bash title="Delete cert-manager CRDs"
kubectl delete crd -l app.kubernetes.io/name=cert-manager
```

### 1.6. Remove stale webhook configs

```bash title="Delete cert-manager webhooks"
kubectl delete validatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook || true
kubectl delete mutatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook || true
```

## 2. Install Cert-Manager

### 2.1. Add repo and install with CRDs

```bash title="Install cert-manager"
helm repo add jetstack https://charts.jetstack.io && helm repo update
helm upgrade --install cert-manager jetstack/cert-manager \
  -n cert-manager --create-namespace \
  --version v1.14.5 \
  --set installCRDs=true
```

### 2.2. Basic health checks

```bash title="Check cert-manager health"
kubectl -n cert-manager get pods
kubectl -n cert-manager logs deploy/cert-manager --tail=200
kubectl get apiservices | grep cert-manager
```

## 3. Configure and Verify TLS

### 3.1. Install staging ClusterIssuer

```bash title="Install staging ClusterIssuer"
kubectl apply -f infra/manifests/cert-manager/clusterissuers.yaml
kubectl get clusterissuer letsencrypt-staging -o yaml
```

### 3.2. Verify HTTP-01 reachability

Ensure your frontend Ingress serves HTTP on port 80 without forced redirect to HTTPS during challenge.

```bash title="Verify HTTP-01 reachability"
curl -I http://dev.<your-domain>/.well-known/acme-challenge/test
```

### 3.3. Annotate Ingress for certificate

Ensure frontend Ingress has the correct annotations and TLS configuration.

### 3.4. Promote to production issuer

Switch annotation to `cert-manager.io/cluster-issuer: letsencrypt-prod` and apply.

## 4. Verification

- `kubectl -n cert-manager get pods` shows all pods are `Running`.
- `kubectl get clusterissuer` shows `Ready=True`.
- `kubectl -n frontend-dev get certificate` shows `Ready=True`.
- The website has a valid HTTPS certificate.
