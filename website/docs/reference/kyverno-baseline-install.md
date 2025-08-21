---
id: kyverno-baseline-install
slug: /reference/kyverno-baseline-install
sidebar_position: 12
---

# Kyverno Baseline Install Guide

:::info This document provides a step-by-step guide for installing the `kyverno` Helm chart and baseline policies. :::

<details>
  <summary>KYV-001 — Add Helm repo and update index</summary>

The first step is to add the `kyverno` Helm repository and update the local Helm repo index.

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
```

After running these commands, you can verify that the `kyverno` chart is available by running the following command:

```bash
helm search repo kyverno/kyverno
```

You should see output similar to this:

```
NAME          	CHART VERSION	APP VERSION	DESCRIPTION
kyverno/kyverno	3.2.4        	v1.12.4    	Kubernetes Native Policy Management
```

</details>

<details>
  <summary>KYV-002 — Install Kyverno with repo values</summary>

Now, install the `kyverno` chart using the provided values file.

```bash
helm upgrade --install kyverno kyverno/kyverno -n kyverno --create-namespace
```

After the installation is complete, you can check the status of the pods in the `kyverno` namespace:

```bash
kubectl get pods -n kyverno
```

All pods should be in the `Running` state and `Ready`.

:::tip Troubleshooting If the Kyverno pods are in a `Pending` state, you can follow these steps to troubleshoot the issue:

1.  **Check the pod description:**

    ```bash
    kubectl describe pod <pod-name> -n kyverno
    ```

    If the event message is `0/2 nodes are available: 2 Too many pods`, it means that the cluster is at its pod capacity.

2.  **Check the node status:** To confirm that the nodes are at their pod capacity, you can run the following command:

    ```bash
    kubectl describe nodes
    ```

    In the output, you will see the `Allocated resources` section for each node. If the number of pods is equal to the `pods` capacity, then the node cannot schedule new pods.

3.  **Scale down non-essential deployments:** To free up resources, you can scale down non-essential deployments. First, list all the pods in the cluster to identify which deployments can be scaled down: `bash kubectl get pods --all-namespaces ` In our case, we identified that the `frontend-sample`, `backend-sample`, and `loki` deployments could be scaled down. Use the following commands to scale down the deployments: `bash kubectl scale deployment frontend-sample -n frontend-dev --replicas=0 kubectl scale deployment backend-sample -n backend-dev --replicas=0 kubectl scale statefulset loki -n loki --replicas=0 ` After scaling down the deployments, the Kyverno pods should be scheduled and running. :::

:::warning Temporary exemptions for networking add-ons When installing a NetworkPolicy engine (e.g., Calico or Cilium), you may need to temporarily relax Kyverno baseline policies to allow the operator/controllers to start:

- Set selected policies (e.g., `require-resource-limits`, `require-run-as-non-root`, signature verification) to `audit` for the duration of the install, or
- Add namespace exclusions for `calico-system` and/or `calico-apiserver` (or the equivalent Cilium namespaces).

After the networking components are Ready, revert the policies to `enforce`. Document any permanent exclusions with rationale. :::

</details>

<details>
  <summary>KYV-003 — Apply baseline policies</summary>

Next, apply the baseline Kyverno policies.

First, review the policies in `infra/manifests/kyverno-policies/policies.yaml`. The policies are:

- `require-image-registry`: Requires images to be from an allowed registry (`ghcr.io`).
- `require-resource-limits`: Requires pods to have CPU and memory resource requests and limits.
- `require-run-as-non-root`: Requires pods to run as a non-root user.
- `verify-image-signatures`: Verifies image signatures for images from `ghcr.io/mpo/web-hosting/*`.

Make sure to add the `kyverno` namespace to the exclusion list in each policy.

Then, apply the policies:

```bash
kubectl apply -f infra/manifests/kyverno-policies/policies.yaml
```

After applying the policies, you can verify that the `ClusterPolicy` objects are created:

```bash
kubectl get clusterpolicies
```

You should see the policies in the output with a `READY` status of `True`.

</details>

<details>
  <summary>KYV-004 — Verify enforcement</summary>

To verify that the policies are being enforced, you can create a test pod that violates the policies.

1.  **Create a test namespace:**

    ```bash
    kubectl create namespace test-kyverno
    ```

2.  **Create a test pod manifest:** Create a file named `test-pod.yaml` with the following content:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pod
    spec:
      containers:
        - name: nginx
          image: nginx
    ```

3.  **Apply the test pod manifest:**

    ```bash
    kubectl apply -f test-pod.yaml -n test-kyverno
    ```

    The command should fail with an error message indicating that the pod was blocked by the `require-image-registry`, `require-resource-limits`, and `require-run-as-non-root` policies.

    This confirms that the policies are being enforced.

4.  **Clean up the test resources:** `bash rm test-pod.yaml kubectl delete namespace test-kyverno `
</details>

<details>
  <summary>KYV-005 — (Optional) Verify image signature policy</summary>

The `verify-image-signatures` policy is included in the baseline policies. However, the public key in the policy is a placeholder. To avoid blocking the pipeline, you can change the `validationFailureAction` from `enforce` to `audit`.

1.  **Update the policy:** In `infra/manifests/kyverno-policies/policies.yaml`, change the `validationFailureAction` for the `verify-image-signatures` policy to `audit` and add `mutateDigest: false` to the `verifyImages` rule:

    ```yaml
    apiVersion: kyverno.io/v1
    kind: ClusterPolicy
    metadata:
      name: verify-image-signatures
    spec:
      validationFailureAction: audit
      rules:
        - name: check-image-signatures
          match:
            any:
              - resources:
                  kinds:
                    - Pod
          exclude:
            any:
              - resources:
                  namespaces:
                    - kube-system
                    - cert-manager
                    - loki
                    - prometheus
                    - ingress
                    - gitops
                    - argocd
                    - kyverno
          verifyImages:
            - image: 'ghcr.io/mpo/web-hosting/*'
              mutateDigest: false
              key: |
                -----BEGIN PUBLIC KEY-----
                MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE...
                -----END PUBLIC KEY-----
    ```

2.  **Apply the updated policies:**
    ```bash
    kubectl apply -f infra/manifests/kyverno-policies/policies.yaml
    ```

This will allow the pipeline to proceed, and a proper key can be added later.

</details>

:::note Network Policy Integration For details on how these Kyverno policies interact with the Calico NetworkPolicies, see the [Network Policies](./network-policies.md) documentation. :::
