---
id: kyverno-revert-001-tighten-kyverno
slug: /archive/network-policies-tasks/kyverno-revert-001-tighten-kyverno
sidebar_position: 4
---

# KYVERNO-REVERT-001 — Tighten Kyverno baseline post-install

- **Status**: Planned
- **Dependencies**: NPOL-VALIDATE-001

## Steps

- Revisit `infra/manifests/kyverno-policies/policies.yaml` and restore intended enforcement after validating network policies.
- Restrict allowed registries if feasible; confirm rollouts remain green.

## Acceptance

- Kyverno ClusterPolicies show intended `validationFailureAction`; new rollouts succeed without violations.
