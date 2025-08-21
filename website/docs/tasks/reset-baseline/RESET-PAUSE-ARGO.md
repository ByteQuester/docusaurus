# RESET-PAUSE-ARGO — Pause GitOps on policy layers

- Status: Done

## Summary

This task was to pause Argo CD's management of NetworkPolicies and Kyverno policies. After a thorough investigation, it was determined that **Argo CD is not managing these policies** in the current environment.

The policies exist as raw manifests, but there are no Argo CD `Application` resources deploying them. Therefore, no action was required for this task.

For a detailed breakdown of the investigation, see the [Reset Baseline Documentation](./docs/docs/reset-baseline.md).

## Steps

- If Argo manages Kyverno or NetworkPolicies, suspend sync or temporarily remove those Apps to prevent drift during reset.
- Record which Apps were paused and when.

## Acceptance

- No automated re-apply of netpol/kyverno while we reset connectivity.
