---
id: kyverno-baseline
slug: /reference/kyverno-baseline
sidebar_position: 13
---

# Kyverno Baseline Policies (Atomic Tasks)

:::info Goal: Complete AWSZ-027 by applying baseline Kyverno policies (registry allowlist, resource limits, non-root, optional image signature verification) and verifying enforcement. :::

## KYV-001 — Install Kyverno

- **Dependencies:** AWSZ-006
- **Steps:**
  - Add Kyverno Helm repo: `helm repo add kyverno https://kyverno.github.io/kyverno/`
  - `helm repo update`
  - Install Kyverno:
    ```bash
    helm upgrade --install kyverno kyverno/kyverno -n kyverno --create-namespace
    ```
- **Acceptance:** `kubectl get pods -n kyverno` shows Kyverno controller/webhook pods Ready.

## KYV-002 — Apply baseline policies

- **Dependencies:** KYV-001
- **Steps:**
  - Review and tune namespaces in `infra/manifests/kyverno-policies/policies.yaml` exclusions
  - Apply policies:
    ```bash
    kubectl apply -f infra/manifests/kyverno-policies/policies.yaml
    ```
- **Acceptance:** `ClusterPolicy` objects created; Kyverno shows policies loaded in logs.

## KYV-003 — Verify enforcement (resource limits)

- **Dependencies:** KYV-002
- **Steps:**
  - Attempt to apply a Pod manifest missing requests/limits; expect validation to fail
  - Example quick test (dry run or temp namespace)
- **Acceptance:** Kyverno denies non-compliant Pod with clear message.

## KYV-004 — Verify registry allowlist

- **Dependencies:** KYV-002
- **Steps:**
  - Attempt to run an image not from `ghcr.io/*` (e.g., `nginx` directly); expect denial
- **Acceptance:** Kyverno denies with the allowlist policy message.

## KYV-005 — Optional: verify image signature policy

- **Dependencies:** KYV-002
- **Steps:**
  - Ensure the public key in `verify-image-signatures` rule is correct; if not, update to your actual signature key or disable this rule for now
- **Acceptance:** Either verification works with your images or the rule is deliberately disabled until keys are set up.
