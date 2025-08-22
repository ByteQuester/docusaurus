---
id: reset-baseline-documentation
slug: /archive/reset-baseline-tasks/reset-baseline-documentation
sidebar_position: 2
---

# Reset Baseline Documentation

This document records the steps and findings of the baseline reset process.

## RESET-PAUSE-ARGO — Pause GitOps on policy layers

**Objective:** If Argo manages Kyverno or NetworkPolicies, suspend sync or temporarily remove those Apps to prevent drift during reset.

### Investigation

The first step was to determine if Argo CD was managing any NetworkPolicies or Kyverno policies. The task description is conditional, suggesting this might not always be the case.

I performed the following actions:

1.  **Searched for `NetworkPolicy` resources in the Helm charts:** This was to check if the application charts included network policies.

    ```bash
    search_file_content(path = "/home/mpo/web/hosting/charts/", pattern = "kind: NetworkPolicy")
    ```

    **Result:** No matches found.

2.  **Searched for Argo CD Applications managing network policies:** I searched for Argo CD `Application` manifests that referenced `network-policies` in their source path.

    ```bash
    search_file_content(path = "/home/mpo/web/hosting/infra/gitops/apps/", pattern = "network-policies")
    ```

    **Result:** No matches found.

3.  **Searched for Argo CD Applications managing Kyverno policies:** I searched for Argo CD `Application` manifests that referenced `kyverno` in their source path.

    ```bash
    search_file_content(path = "/home/mpo/web/hosting/infra/", pattern = "kyverno")
    ```

    **Result:** Found Kyverno policies in `infra/manifests/kyverno-policies/`, but no Argo CD Application managing them.

4.  **Listed all Argo CD Applications from the cluster:** To get a definitive list of all applications managed by Argo CD, I used `kubectl`.

    ```bash
    kubectl get applications -n argocd -o yaml --kubeconfig=$HOME/.kube/config
    ```

    **Result:** The output confirmed that only the `root`, `frontend-sample`, and `backend-sample` applications are managed by Argo CD.

### Conclusion

Based on the investigation, **Argo CD is not managing NetworkPolicies or Kyverno policies in this environment.** The policies exist as raw manifests in `infra/manifests/` but are not deployed or managed by Argo CD.

Therefore, the task of pausing Argo's management of these policies is **not applicable**. No action is required.

## RESET-NETPOL-CLEAR — Remove all NetworkPolicies (temporary)

**Objective:** Remove all `NetworkPolicy` resources from all namespaces to temporarily allow all traffic during connectivity tests.

### Execution

I executed the following command to delete all network policies in the cluster:

```bash
kubectl delete netpol --all -A --kubeconfig=$HOME/.kube/config || true
```

**Result:**

```
networkpolicy.networking.k8s.io "argocd-application-controller-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-applicationset-controller-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-dex-server-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-notifications-controller-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-redis-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-repo-server-network-policy" deleted
networkpolicy.networking.k8s.io "argocd-server-network-policy" deleted
networkpolicy.networking.k8s.io "15-allow-from-ingress" deleted
networkpolicy.networking.k8s.io "allow-dns" deleted
networkpolicy.networking.k8s.io "allow-from-frontend" deleted
networkpolicy.networking.k8s.io "default-deny-all" deleted
networkpolicy.networking.k8s.io "allow-dns" deleted
networkpolicy.networking.k8s.io "allow-from-default-for-debug" deleted
networkpolicy.networking.k8s.io "allow-from-ingress" deleted
networkpolicy.networking.k8s.io "allow-ingress-webhook" deleted
networkpolicy.networking.k8s.io "allow-to-backend-dev-namespace" deleted
networkpolicy.networking.k8s.io "default-deny-all" deleted
networkpolicy.networking.k8s.io "allow-from-prometheus" deleted
networkpolicy.networking.k8s.io "allow-egress-to-ingress-webhook" deleted
networkpolicy.networking.k8s.io "external-dns" deleted
```

### Verification

To verify that all network policies were deleted, I ran the following command:

```bash
kubectl get netpol -A --kubeconfig=$HOME/.kube/config
```

**Result:**

```
No resources found
```

This confirms that all network policies have been successfully removed.
