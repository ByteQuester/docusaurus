# DNS-TRIAGE-001 — Control plane connectivity

- Status: In Progress
- Dependencies: AWSZ-006

## Scope

Confirm that nodes/pods can reach the Kubernetes API server; ensure kube-proxy is healthy and Service IP `kubernetes.default.svc` resolves/routs.

## Commands

```bash
kubectl get pods -n kube-system --kubeconfig=$HOME/.kube/config | egrep "kube-proxy|coredns" | cat
kubectl logs -n kube-system -l k8s-app=kube-proxy --kubeconfig=$HOME/.kube/config --tail=200 | cat || true

# From a debug pod in kube-system, test API server via service and endpoint
kubectl apply -f debug-pod-kube-system.yaml --kubeconfig=$HOME/.kube/config
kubectl exec -n kube-system debug-kube-system --kubeconfig=$HOME/.kube/config -- sh -c "wget -qO- https://kubernetes.default.svc" || true
kubectl exec -n kube-system debug-kube-system --kubeconfig=$HOME/.kube/config -- sh -c "wget -qO- https://$$(kubectl get ep kubernetes -n default -o jsonpath='{.subsets[0].addresses[0].ip}'):443" || true
```

## Acceptance

- kube-proxy pods Ready; no fatal errors.
- In-cluster HTTPS to API server via ServiceIP and endpoint succeeds (TLS likely fails to show page but connection should establish).
