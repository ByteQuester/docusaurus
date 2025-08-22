---
id: minimal-connectivity-reset
slug: /runbooks/minimal-connectivity-reset
sidebar_position: 9
---

# Minimal Connectivity Reset

:::info This runbook provides a set of atomic tasks to unblock development by resetting to a working baseline: no custom NetworkPolicies, simple RBAC (chart-managed SAs), default CNI settings, and verified DNS/API connectivity. :::

## 1. Pause GitOps on Policy Layers

**Goal:** If Argo manages Kyverno or NetworkPolicies, suspend sync or temporarily remove those Apps to prevent drift during reset.

**Action:**

- Investigate if Argo CD is managing any NetworkPolicies or Kyverno policies.
- If so, suspend sync or temporarily remove the corresponding Argo CD Applications.

## 2. Remove all NetworkPolicies

**Goal:** Remove all `NetworkPolicy` resources from all namespaces to temporarily allow all traffic during connectivity tests.

**Action:**

```bash title="Delete all NetworkPolicies"
kubectl delete netpol --all -A
```

## 3. Revert aws-node and CNI changes

**Goal:** Restore default AWS VPC CNI settings; remove experimental toggles intended to "enable network policy" in `aws-node`.

**Action:**

- Inspect the `aws-node` DaemonSet in the `kube-system` namespace for any non-default environment variables and revert them.

## 4. Switch to chart-managed ServiceAccounts

**Goal:** For early templates, prefer chart-managed ServiceAccounts to reduce naming drift.

**Action:**

- Update chart values to enable chart-managed ServiceAccounts.
- Remove any conflicting precreated ServiceAccounts or RoleBindings.
- Sync via Argo/Helm and verify pods use chart-created SAs.

## 5. Validate Core Cluster Functionality

**Goal:** Validate core cluster functionality after the reset.

**Actions:**

- **API reachability:**
  ```bash
  kubectl exec -n default debug -- sh -c "wget -qO- https://kubernetes.default.svc"
  ```
- **DNS resolution:**
  ```bash
  kubectl exec -n default debug -- nslookup kubernetes.default.svc
  ```
- **Service routing:**
  ```bash
  kubectl exec -n frontend-dev debug -- curl -sv http://backend-sample.backend-dev:80/api/health
  ```

## 6. Document Decisions and Templates

**Goal:** Document the reset decision and the temporary simplifications.

**Action:**

- Update relevant documentation to reflect the changes made during this reset process.
