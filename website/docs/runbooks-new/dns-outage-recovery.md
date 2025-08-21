---
id: dns-outage-recovery
slug: /runbooks/dns-outage-recovery
sidebar_position: 7
---

# DNS Outage Recovery (Atomic Tasks)

:::info This runbook provides a set of atomic tasks to restore cluster DNS and website availability by resolving CoreDNS → API server connectivity and standardizing network policy/debug artifacts. :::

## Subtask Index

- [DNS-TRIAGE-001 — Control plane connectivity](../../../tasks/dns-outage/DNS-TRIAGE-001_control-plane-connectivity.md)
- [DNS-TRIAGE-002 — CoreDNS health and config](../../../tasks/dns-outage/DNS-TRIAGE-002_coredns-health.md)
- [DNS-TRIAGE-003 — CNI and NetworkPolicy engine alignment](../../../tasks/dns-outage/DNS-TRIAGE-003_cni-netpol-alignment.md)
- [DNS-FIX-001 — Cleanup stray files and temporary policies](../../../tasks/dns-outage/DNS-FIX-001_cleanup-stray-files.md)
- [DNS-FIX-002 — EKS endpoint and security groups validation](../../../tasks/dns-outage/DNS-FIX-002_eks-endpoint-and-sg.md)
- [DNS-FIX-003 — kube-proxy and ClusterIP path validation](../../../tasks/dns-outage/DNS-FIX-003_kubeproxy-clusterip.md)
- [DNS-VALIDATE-001 — Validate DNS and site end-to-end](../../../tasks/dns-outage/DNS-VALIDATE-001_end-to-end.md)
- [DNS-DOCS-001 — Update debugging report and docs](../../../tasks/dns-outage/DNS-DOCS-001_update-docs.md)

:::tip Notes

- Always use `--kubeconfig=$HOME/.kube/config` for commands.
- Prefer verification evidence in task files, not chat. Keep `../../../tasks/task_log.md` as pointers. :::
