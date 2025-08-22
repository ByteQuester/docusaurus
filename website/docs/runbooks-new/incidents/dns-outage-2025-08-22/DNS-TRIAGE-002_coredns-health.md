# DNS-TRIAGE-002 — CoreDNS health and config

- Status: In Progress
- Dependencies: DNS-TRIAGE-001

## Commands

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns --kubeconfig=$HOME/.kube/config -o wide | cat
kubectl describe pod -n kube-system -l k8s-app=kube-dns --kubeconfig=$HOME/.kube/config | cat
kubectl logs -n kube-system -l k8s-app=kube-dns --kubeconfig=$HOME/.kube/config --tail=200 | cat
kubectl get svc -n kube-system kube-dns --kubeconfig=$HOME/.kube/config -o wide | cat
kubectl get cm -n kube-system coredns --kubeconfig=$HOME/.kube/config -o yaml | cat
```

## Checks

- Pods Ready; readiness probes pass.
- `kube-dns` Service ClusterIP present and reachable from app namespaces.
- CoreDNS ConfigMap has upstream settings intact; no customizations blocking API calls.

## Acceptance

- CoreDNS pods Ready; no recurring dial timeouts to API server; DNS lookup from a debug pod succeeds (tested in DNS-VALIDATE-001).
