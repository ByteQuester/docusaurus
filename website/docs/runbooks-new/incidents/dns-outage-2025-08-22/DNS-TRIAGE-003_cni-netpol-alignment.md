# DNS-TRIAGE-003 — CNI and NetworkPolicy engine alignment

- Status: In Progress
- Dependencies: NPOL-000, NPOL-TRIAGE-001

## Context

EKS uses AWS VPC CNI for pod networking. NetworkPolicy enforcement requires an engine (Calico or Cilium). Do not attempt to "enable" NetworkPolicies in `aws-node`; instead, ensure Calico is present and kube-proxy is healthy.

## Commands

```bash
kubectl get pods -n tigera-operator,calico-system --kubeconfig=$HOME/.kube/config | cat || true
kubectl get netpol -A --kubeconfig=$HOME/.kube/config | cat
kubectl get netpol -n kube-system --kubeconfig=$HOME/.kube/config | cat
```

## Checks

- Calico components Ready.
- No deny policies in `kube-system` that would block CoreDNS egress to API server.
- No conflicting attempts to configure NetworkPolicy in `aws-node`.

## Acceptance

- Single enforcement engine confirmed (Calico). `kube-system` has no restrictive policies blocking DNS or API connectivity.
