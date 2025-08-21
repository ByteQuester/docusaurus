---
id: capacity-audit
slug: /reference/capacity-audit
sidebar_position: 3
---

# Capacity Audit

:::info This document provides a summary of the cluster-wide audit for resource requests and limits on deployments. :::

## CAP-003 — Requests/limits audit

As part of the capacity baseline initiative, an audit was performed to identify all deployments that are missing resource requests and/or limits.

### Findings

The following deployments were found to be missing requests and/or limits:

- **argocd**:
  - `argocd-applicationset-controller`
  - `argocd-dex-server`
  - `argocd-notifications-controller`
  - `argocd-redis`
  - `argocd-repo-server`
  - `argocd-server`
- **calico-apiserver**:
  - `calico-apiserver`
- **calico-system**:
  - `calico-kube-controllers`
  - `calico-typha`
  - `goldmane`
  - `whisker`
- **cert-manager**:
  - `cert-manager`
  - `cert-manager-cainjector`
  - `cert-manager-webhook`
- **ingress**:
  - `ingress-nginx-controller` (missing limits)
- **kube-system**:
  - `cluster-autoscaler-aws-cluster-autoscaler`
  - `coredns` (missing cpu limit)
  - `metrics-server` (missing limits)
- **kyverno**:
  - `kyverno-admission-controller` (missing cpu limit)
  - `kyverno-background-controller` (missing cpu limit)
  - `kyverno-cleanup-controller` (missing cpu limit)
  - `kyverno-reports-controller` (missing cpu limit)
- **prometheus**:
  - `blackbox-exporter-prometheus-blackbox-exporter`
  - `prometheus-grafana`
  - `prometheus-kube-prometheus-operator`
  - `prometheus-kube-state-metrics`

:::tip Recommendations It is recommended to set appropriate resource requests and limits for all of the above deployments to improve cluster stability and resource management. This will also allow for more accurate capacity planning.

Follow-up tasks should be created to address these findings. :::
