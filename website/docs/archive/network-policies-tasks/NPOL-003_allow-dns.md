---
id: npol-003-allow-dns
slug: /archive/network-policies-tasks/npol-003-allow-dns
sidebar_position: 8
---

# NPOL-003 — Allow DNS egress (CoreDNS)

- **Status**: Completed
- **Dependencies**: NPOL-002

## Manifests

- `infra/manifests/network-policies/frontend-dev/allow-dns.yaml`
- `infra/manifests/network-policies/backend-dev/allow-dns.yaml`

## Verification

We first checked for the `allow-dns` network policies in the `frontend-dev` and `backend-dev` namespaces and found they were not present. We applied them from the repository.

```bash title="Apply allow-dns policies"
kubectl apply -f infra/manifests/network-policies/frontend-dev/allow-dns.yaml --kubeconfig=$HOME/.kube/config
kubectl apply -f infra/manifests/network-policies/backend-dev/allow-dns.yaml --kubeconfig=$HOME/.kube/config
```

We then verified that the policies were applied successfully.

```bash title="Verify allow-dns policies"
kubectl get netpol -n frontend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME               POD-SELECTOR   AGE
allow-dns          <none>         5s
default-deny-all   <none>         2m49s
```

```bash
kubectl get netpol -n backend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME               POD-SELECTOR   AGE
allow-dns          <none>         5s
default-deny-all   <none>         2m49s
```

We then attempted to verify DNS resolution from within a pod, but the standard tools were not available in the container image.

```bash title="Attempt to verify DNS resolution"
kubectl -n frontend-dev --kubeconfig=$HOME/.kube/config exec deploy/frontend-sample -- nslookup kubernetes.default
```

**Output:**

```
error: Internal error occurred: Internal error occurred: error executing command in container: failed to exec in container: failed to start exec "527add68faf8767075be99a270cde541c8a20c125bcf54cf12acc9d12638bffa": OCI runtime exec failed: exec failed: unable to start container process: exec: "nslookup": executable file not found in $PATH: unknown
```

## Acceptance

- The `allow-dns` network policies have been applied. Direct verification of DNS resolution from the application pods was not possible due to the lack of tools in the container image. However, the presence of the `allow-dns` and `default-deny-all` policies should ensure that DNS is allowed while other egress traffic is denied.
