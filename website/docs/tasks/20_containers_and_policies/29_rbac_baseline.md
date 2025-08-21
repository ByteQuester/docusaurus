---
id: rbac-baseline
slug: /tasks/20-containers-and-policies/rbac-baseline
sidebar_position: 3
---

# RBAC Baseline

:::info **Goal:** Enforce least-privilege access for app workloads; bind ServiceAccounts to minimal Roles. :::

## Subtask Index

- [RBAC-INV-001 — Inventory current RBAC](./rbac/RBAC-INV-001_inventory.md)
- [RBAC-ALIGN-001 — Align chart ServiceAccounts with cluster SAs](./rbac/RBAC-ALIGN-001_chart-serviceaccounts.md)
- [RBAC-APPLY-001 — Apply RBAC manifests](./rbac/RBAC-APPLY-001_apply-manifests.md)
- [RBAC-VERIFY-001 — Verify RoleBindings and pod ServiceAccounts](./rbac/RBAC-VERIFY-001_verify-bindings.md)
- [RBAC-TEST-001 — Runtime behavior checks (no forbidden errors)](./rbac/RBAC-TEST-001_runtime-behavior.md)
- [RBAC-DOCS-001 — Update security docs](./rbac/RBAC-DOCS-001_update-docs.md)
- [RBAC-ROLLBACK-001 — Rollback plan](./rbac/RBAC-ROLLBACK-001_plan.md)

---

### RBAC-000 — Inventory current RBAC and ServiceAccounts

- **Status:** Completed
- **Dependencies:** AWSZ-006
- **Steps:**
  - List Roles/RoleBindings and ServiceAccounts in `frontend-dev` and `backend-dev`.
  - Map Deployments → ServiceAccounts → RoleBindings.
- **Commands:**
  ```bash
  kubectl get sa,role,rolebinding -n frontend-dev --kubeconfig=$HOME/.kube/config
  kubectl get sa,role,rolebinding -n backend-dev --kubeconfig=$HOME/.kube/config
  ```
- **Findings:**
  - **frontend-dev:**
    - ServiceAccounts: `default`, `frontend`, `frontend-sample`
    - Roles: none
    - RoleBindings: none
    - Deployment `frontend-sample` uses ServiceAccount `frontend-sample`.
  - **backend-dev:**
    - ServiceAccounts: `backend-sample`, `default`
    - Roles: none
    - RoleBindings: none
    - Deployment `backend-sample` uses ServiceAccount `backend-sample`.
- **Acceptance:** Inventory captured in this file; gaps identified.

### RBAC-001 — Define minimal Roles

- **Status:** Completed
- **Dependencies:** RBAC-000
- **Manifests:**
  - `infra/manifests/rbac/frontend-rbac.yaml`
  - `infra/manifests/rbac/backend-rbac.yaml`
- **Verification:**

  - We applied the RBAC manifests and verified that the roles and rolebindings were created.

  **frontend-dev:**

  ```bash
  kubectl get role,rolebinding -n frontend-dev --kubeconfig=$HOME/.kube/config
  ```

  **Output:**

  ```
  NAME                                           CREATED AT
  role.rbac.authorization.k8s.io/frontend-role   2025-08-20T08:14:23Z

  NAME                                                         ROLE                 AGE
  rolebinding.rbac.authorization.k8s.io/frontend-rolebinding   Role/frontend-role   117s
  ```

  **backend-dev:**

  ```bash
  kubectl get role,rolebinding -n backend-dev --kubeconfig=$HOME/.kube/config
  ```

  **Output:**

  ```
  NAME                                                 CREATED AT
  role.rbac.authorization.k8s.io/backend-sample-role   2025-08-20T08:16:20Z

  NAME                                                               ROLE                       AGE
  rolebinding.rbac.authorization.k8s.io/backend-sample-rolebinding   Role/backend-sample-role   19s
  ```

- **Acceptance:** Roles/Bindings present; pods run without RBAC errors.

### RBAC-002 — Wire Deployments to ServiceAccounts

- **Status:** Completed
- **Dependencies:** RBAC-001
- **Steps:**
  - We have verified that the `frontend-sample` and `backend-sample` deployments are already using the correct ServiceAccounts (`frontend-sample` and `backend-sample` respectively).
  - No rollout restart is needed.
- **Acceptance:** `kubectl describe deploy` shows correct ServiceAccount; pods Ready.

### RBAC-003 — Document RBAC in docs

- **Status:** Completed
- **Dependencies:** RBAC-001
- **Steps:**
  - We have updated `../security.md` with a summary of the RBAC strategy and links to the manifests.
- **Acceptance:** Documentation updated and cross-linked.

---

:::note Deliverables

- Minimal Role/RoleBinding manifests applied; ServiceAccounts bound.
- Verification output captured in this file.
- Cross-links added to docs. :::

:::warning Notes

- There is a ServiceAccount naming mismatch risk between Helm charts and standalone RBAC manifests. Prefer one source of truth: - Set charts to `serviceAccount.create: false` and `serviceAccount.name` to the precreated SA from manifests (recommended), - Or modify RoleBindings to reference the chart-created SA names. Capture the chosen approach in `RBAC-ALIGN-001`. :::
