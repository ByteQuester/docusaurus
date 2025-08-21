# RESET-RBAC-SIMPLE — Switch to chart-managed ServiceAccounts

- Status: In Progress

## Decision

For early templates, prefer chart-managed ServiceAccounts to reduce naming drift. We will switch back to manifest-managed later if needed.

## Steps

- Update chart values:

```yaml
# charts/frontend-sample/values.yaml
serviceAccount:
  create: true
  name: ""

# charts/backend-sample/values.yaml
serviceAccount:
  create: true
  name: ""
```

- Remove any conflicting precreated ServiceAccounts or RoleBindings that point to different names (keep Roles empty/minimal).
- Sync via Argo/Helm; verify pods use chart-created SAs.

## Acceptance

- Pods show `.spec.serviceAccountName` aligned with chart defaults (`frontend-sample`, `backend-sample`); apps run without RBAC errors.

## Roll-forward note

- Later, when templating for production, we can flip to manifest-managed SAs again with a clear, single naming convention.
