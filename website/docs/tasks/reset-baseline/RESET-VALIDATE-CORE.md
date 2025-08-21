# RESET-VALIDATE-CORE — Validate DNS, API, ClusterIP routing

- Status: Planned

## API reachability (in-cluster)

```bash
kubectl apply -f ops/debug/debug-pod.yaml --kubeconfig=$HOME/.kube/config || true
kubectl exec -n default debug --kubeconfig=$HOME/.kube/config -- sh -c "wget -qO- https://kubernetes.default.svc" || true
```

## DNS resolution

```bash
kubectl exec -n default debug --kubeconfig=$HOME/.kube/config -- nslookup kubernetes.default.svc || getent hosts kubernetes.default.svc || true
kubectl exec -n frontend-dev debug --kubeconfig=$HOME/.kube/config -- nslookup backend-sample.backend-dev.svc || getent hosts backend-sample.backend-dev.svc || true
```

## Service routing

```bash
kubectl exec -n frontend-dev debug --kubeconfig=$HOME/.kube/config -- curl -sv http://backend-sample.backend-dev:80/api/health | cat
```

## Acceptance

- API reachable via ServiceIP; DNS resolves; ClusterIP routing works across namespaces.
