# NPOL-TRIAGE-003 — Verify allow rules for backend ingress and frontend

- Status: Completed
- Dependencies: NPOL-004, NPOL-005

## Steps

We verified that the `allow-from-frontend` and `15-allow-from-ingress` network policies are applied in the `backend-dev` namespace.

```bash
kubectl get netpol -n backend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME                    POD-SELECTOR                            AGE
15-allow-from-ingress   app.kubernetes.io/name=backend-sample   4s
allow-dns               <none>                                  6m53s
allow-from-frontend     app.kubernetes.io/name=backend-sample   5m16s
default-deny-all        <none>                                  9m37s
```

We also verified connectivity from the ingress to the backend, and attempted to verify connectivity from the frontend to the backend.

- **Ingress to Backend:** Successful, as documented in `NPOL-005`.
- **Frontend to Backend:** Could not be verified directly due to a lack of tools in the frontend container, as documented in `NPOL-004`.

## Acceptance

- Both allow policies are present with the correct selectors.
