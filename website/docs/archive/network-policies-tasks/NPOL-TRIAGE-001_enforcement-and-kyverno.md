---
id: npol-triage-001-enforcement-and-kyverno
slug: /archive/network-policies-tasks/npol-triage-001-enforcement-and-kyverno
sidebar_position: 15
---

# NPOL-TRIAGE-001 — Confirm enforcement engine and Kyverno relaxations

- **Status**: In Progress
- **Dependencies**: AWSZ-006, NPOL-000

## Steps

```bash title="Check Calico and Kyverno policies"
kubectl get pods -n tigera-operator,calico-system --kubeconfig=$HOME/.kube/config | cat
kubectl get cpol --kubeconfig=$HOME/.kube/config | cat || true
```

Verify Kyverno policies and `validationFailureAction` settings align with current rollout stage.

## Acceptance

- Calico components are Ready.
- Kyverno policies configured as intended for baseline rollout.
