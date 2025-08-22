---
id: npol-008-apply-validate-rollback
slug: /archive/network-policies-tasks/npol-008-apply-validate-rollback
sidebar_position: 14
---

# NPOL-008 — Apply order, tests, and rollback

- **Status**: Completed
- **Dependencies**: NPOL-003..NPOL-007

## Apply order

The following policies were applied:

**backend-dev:**

1.  `00-deny-all.yaml`
2.  `allow-dns.yaml`
3.  `10-allow-from-frontend.yaml`
4.  `15-allow-from-ingress.yaml`

**frontend-dev:**

1.  `00-deny-all.yaml`
2.  `allow-dns.yaml`
3.  `15-allow-from-ingress.yaml`
4.  `allow-to-backend-dev-namespace.yaml`

## Validation

Internal and external connectivity was validated successfully after a lengthy troubleshooting process.

**Internal check:**

```bash title="Internal connectivity check"
kubectl exec -it tmp-debug -n frontend-dev --kubeconfig=$HOME/.kube/config -- curl -sv http://backend-sample.backend-dev:80/api/health | cat
```

**External check:**

```bash title="External connectivity check"
curl -sv https://dev.capitabyte.com/api/health
```

## Rollback

To rollback the network policies, run the following commands:

```bash title="Rollback network policies"
kubectl delete -f infra/manifests/network-policies/frontend-dev/
kubectl delete -f infra/manifests/network-policies/backend-dev/
```

**Smoke Test:**

After deleting the policies, the following curl command should succeed:

```bash title="Smoke test after rollback"
kubectl exec -it tmp-debug -n frontend-dev --kubeconfig=$HOME/.kube/config -- curl -v http://backend-sample.backend-dev:80/api/health | cat
```

## Acceptance

- App remains functional under policies; rollback tested and documented in `tasks/task_log.md`.
