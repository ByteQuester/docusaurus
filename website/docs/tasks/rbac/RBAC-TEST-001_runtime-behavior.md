# RBAC-TEST-001 — Runtime behavior checks (no forbidden errors)

- Status: Planned
- Dependencies: RBAC-VERIFY-001

## Steps

- Tail logs of `frontend-sample` and `backend-sample` post-deploy to ensure no RBAC Forbidden errors appear.

```bash
kubectl logs -n frontend-dev deploy/frontend-sample --kubeconfig=$HOME/.kube/config | egrep -i "forbidden|rbac|permission|denied" || true
kubectl logs -n backend-dev deploy/backend-sample --kubeconfig=$HOME/.kube/config | egrep -i "forbidden|rbac|permission|denied" || true
```

## Acceptance

- No runtime permission errors; apps function normally.
