# RESET-AWS-NODE-REVERT — Revert aws-node and CNI changes

- Status: In Progress

## Scope

Restore default AWS VPC CNI settings; remove experimental toggles intended to "enable network policy" in aws-node. Keep Calico as the sole NetworkPolicy engine if present.

## Steps

- Inspect `kube-system` DaemonSet `aws-node` for non-default env vars (e.g., policy toggles) and revert.
- If Tigera operator was added on top of existing Calico, decide to keep the current install or cleanly uninstall the duplicate addition. Do not run dual enforcement.

## Acceptance

- aws-node runs with default settings; Calico pods Ready (single enforcement engine confirmed).
