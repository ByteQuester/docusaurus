# DNS-FIX-003 — kube-proxy and ClusterIP path validation

- Status: In Progress
- Dependencies: DNS-TRIAGE-001

## Steps

- Verify kube-proxy iptables/ipvs mode is consistent and not erroring.

```bash
kubectl logs -n kube-system -l k8s-app=kube-proxy --kubeconfig=$HOME/.kube/config --tail=200 | cat
```

- From a pod in `frontend-dev` with DNS allowed, resolve and curl a ClusterIP (e.g., backend service) to ensure service routing works.

```bash
kubectl exec -n frontend-dev tmp-debug --kubeconfig=$HOME/.kube/config -- getent hosts backend-sample.backend-dev.svc.cluster.local || true
kubectl exec -n frontend-dev tmp-debug --kubeconfig=$HOME/.kube/config -- sh -c 'IP=$(getent hosts backend-sample.backend-dev | awk "{print $1}"); [ -n "$IP" ] && wget -qO- http://$IP:80/api/health || true'
```

## Acceptance

- kube-proxy shows no fatal errors; service routing works for ClusterIP services across namespaces.
