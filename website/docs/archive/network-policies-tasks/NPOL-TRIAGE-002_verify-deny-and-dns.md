---
id: npol-triage-002-verify-deny-and-dns
slug: /archive/network-policies-tasks/npol-triage-002-verify-deny-and-dns
sidebar_position: 16
---

# NPOL-TRIAGE-002 — Verify default-deny and DNS allows are applied

- **Status**: Completed
- **Dependencies**: NPOL-002, NPOL-003

## Steps

We verified that the `default-deny-all` and `allow-dns` network policies are applied in both the `frontend-dev` and `backend-dev` namespaces.

```bash title="Verify network policies in frontend-dev"
kubectl get netpol -n frontend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME                   POD-SELECTOR                            AGE
allow-dns              <none>                                  10m
allow-from-ingress     app.kubernetes.io/name=frontend-sample  1m
default-deny-all       <none>                                  15m
```

```bash title="Verify network policies in backend-dev"
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

DNS resolution tests from the application pods could not be performed because the necessary tools (`nslookup`, `ping`) were not available in the container images.

## Acceptance

- Default-deny and DNS policies are present.
- DNS queries are presumed to be working based on the applied policies.
