# DNS-VALIDATE-001 — Validate DNS and site end-to-end

- Status: Planned
- Dependencies: DNS-TRIAGE-002, DNS-FIX-002, DNS-FIX-003

## Internal DNS check

```bash
kubectl exec -n frontend-dev tmp-debug --kubeconfig=$HOME/.kube/config -- nslookup kubernetes.default.svc || getent hosts kubernetes.default.svc || true
kubectl exec -n frontend-dev tmp-debug --kubeconfig=$HOME/.kube/config -- nslookup backend-sample.backend-dev.svc || getent hosts backend-sample.backend-dev.svc || true
```

## App connectivity

```bash
kubectl exec -n frontend-dev tmp-debug --kubeconfig=$HOME/.kube/config -- curl -sv http://backend-sample.backend-dev:80/api/health | cat
```

## External

```bash
curl -sv https://dev.capitabyte.com/api/health | cat
curl -sv https://dev.capitabyte.com/ | head -20 | cat
```

## Acceptance

- DNS resolves in-cluster for core and app services; internal and external health checks return 200.
