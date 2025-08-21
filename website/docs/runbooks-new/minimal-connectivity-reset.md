---
id: minimal-connectivity-reset
slug: /runbooks/minimal-connectivity-reset
sidebar_position: 9
---

# Minimal Connectivity Reset (Atomic Tasks)

:::info This runbook provides a set of atomic tasks to unblock development by resetting to a working baseline: no custom NetworkPolicies, simple RBAC (chart-managed SAs), default CNI settings, and verified DNS/API connectivity. :::

## Subtask Index

- [RESET-PAUSE-ARGO — Pause GitOps on netpol/kyverno layers](../../../tasks/reset-baseline/RESET-PAUSE-ARGO.md)
- [RESET-NETPOL-CLEAR — Remove all app/kube-system NetworkPolicies](../../../tasks/reset-baseline/RESET-NETPOL-CLEAR.md)
- [RESET-AWS-NODE-REVERT — Revert aws-node and CNI changes](../../../tasks/reset-baseline/RESET-AWS-NODE-REVERT.md)
- [RESET-RBAC-SIMPLE — Switch to chart-managed ServiceAccounts](../../../tasks/reset-baseline/RESET-RBAC-SIMPLE.md)
- [RESET-VALIDATE-CORE — Validate DNS, API, ClusterIP routing](../../../tasks/reset-baseline/RESET-VALIDATE-CORE.md)
- [RESET-DOCS-LINKS — Capture decisions/templates](../../../tasks/reset-baseline/RESET-DOCS-LINKS.md)

:::tip Notes

- Always use `--kubeconfig=$HOME/.kube/config`.
- Keep manifests-only enforcement out of kube-system during reset. :::
