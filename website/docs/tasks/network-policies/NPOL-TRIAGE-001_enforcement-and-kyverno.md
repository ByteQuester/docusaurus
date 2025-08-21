# NPOL-TRIAGE-001 — Confirm enforcement engine and Kyverno relaxations

- Status: In Progress
- Dependencies: AWSZ-006, NPOL-000

## Steps

```bash
kubectl get pods -n tigera-operator,calico-system --kubeconfig=$HOME/.kube/config | cat
kubectl get cpol --kubeconfig=$HOME/.kube/config | cat || true
```

Verify Kyverno policies and `validationFailureAction` settings align with current rollout stage.

## Acceptance

- Calico components are Ready.
- Kyverno policies configured as intended for baseline rollout.
