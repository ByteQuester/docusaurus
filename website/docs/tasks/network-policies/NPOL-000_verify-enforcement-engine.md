# NPOL-000 — Verify NetworkPolicy enforcement engine

- Status: In Progress
- Dependencies: AWSZ-006 (kubectl context)

## Scope

Confirm a policy engine (Calico or Cilium) is installed and enforcing NetworkPolicy; document version and selectors linkage.

## Steps

```bash
kubectl get pods -n tigera-operator,calico-system --kubeconfig=$HOME/.kube/config | cat || true
kubectl get pods -n kube-system --kubeconfig=$HOME/.kube/config | egrep "cilium|calico|kube-dns" | cat || true
```

If not present, install Calico per `docs/docs/calico-install.md`.

## Acceptance

- Calico or Cilium components are Ready.
- A default-deny test is enforceable once applied.

## Links

- `docs/docs/observability.md` — enforcement engine and selectors
- `infra/manifests/network-policies/README.md`
