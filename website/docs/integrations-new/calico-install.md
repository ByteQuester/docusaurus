---
id: calico-install
slug: /integrations/calico-install
sidebar_position: 1
---

# Calico Installation Guide

:::info This document provides a step-by-step guide for installing Calico as a NetworkPolicy enforcement engine. :::

## NPOL-000 — Verify NetworkPolicy enforcement engine

The first step is to install Calico and verify that it is working correctly.

### Initial Installation and Kyverno Policy Modifications

The first step is to install the `tigera-operator` using the Calico Helm repository.

```bash title="Add Calico Helm repository"
helm repo add calico https://docs.tigera.io/calico/charts
helm repo update
```

:::warning Kyverno Policies The initial installation will likely fail due to the restrictive Kyverno policies in place. To allow the installation, you need to temporarily modify the Kyverno `ClusterPolicy` definitions in `infra/manifests/kyverno-policies/policies.yaml`. :::

First, add `quay.io` to the list of allowed image registries in the `require-image-registry` policy:

```yaml title="infra/manifests/kyverno-policies/policies.yaml"
# ...
validate:
  message: 'Images must be from an allowed registry.'
  pattern:
    spec:
      containers:
        - image: 'ghcr.io/*|quay.io/*'
# ...
```

Next, set the `validationFailureAction` to `audit` for the following policies:

| Policy                    | New `validationFailureAction` |
| ------------------------- | ----------------------------- |
| `require-run-as-non-root` | `audit`                       |
| `require-resource-limits` | `audit`                       |

After modifying the policies, apply the changes to the cluster:

```bash title="Apply Kyverno policy changes"
kubectl apply -f infra/manifests/kyverno-policies/policies.yaml
```

Now, you can install the `tigera-operator` Helm chart.

```bash title="Install tigera-operator"
helm install calico calico/tigera-operator --namespace tigera-operator --create-namespace
```

### Create the Calico Installation Custom Resource

After the `tigera-operator` is installed, you need to create a Calico `Installation` custom resource.

:::note For AWS EKS It is important to set `spec.calicoNetwork.ipPools[0].encapsulation` to `None` to use the AWS VPC CNI for the dataplane. :::

Create a file named `calico-installation.yaml` with the following content:

```yaml title="calico-installation.yaml"
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  variant: Calico
  calicoNetwork:
    ipPools:
      - name: default-ipv4-ippool
        cidr: 192.168.0.0/16
        encapsulation: None
        natOutgoing: Enabled
        nodeSelector: all()
```

Then, apply the file to the cluster:

```bash
kubectl apply -f calico-installation.yaml
```

### Troubleshooting Pending Pods and Cluster Resizing

:::tip Troubleshooting After applying the `calico-installation.yaml`, you may encounter issues with Calico pods being stuck in a `Pending` state. This is often due to insufficient resources in the cluster. :::

You can check the status of the Calico pods using the following command:

```bash
kubectl get pods -n calico-system
```

If you see pods in a `Pending` state, you can use `kubectl describe pod <pod-name> -n calico-system` to get more information. If the error message is `Too many pods`, it means that the cluster is too small to handle the Calico installation.

To resolve this issue, you need to resize the EKS cluster nodegroup. You can use the `awscli` to do this.

1.  **Find the name of your nodegroup:**
    ```bash
    aws eks list-nodegroups --cluster-name <your-cluster-name> --region <your-region>
    ```
2.  **Describe the nodegroup to get the current size:**
    ```bash
    aws eks describe-nodegroup --cluster-name <your-cluster-name> --nodegroup-name <your-nodegroup-name> --region <your-region>
    ```
3.  **Update the nodegroup to increase the number of nodes:**
    ```bash
    aws eks update-nodegroup-config --cluster-name <your-cluster-name> --nodegroup-name <your-nodegroup-name> --region <your-region> --scaling-config minSize=1,maxSize=5,desiredSize=5
    ```

After the nodegroup has been resized, the Calico pods should start successfully.

### Verify Installation with a Default-Deny Test

After the Calico pods are running, you should verify that the network policies are being enforced. You can do this by creating a simple default-deny policy and testing connectivity between pods.

1.  **Create a test namespace:**
    ```bash
    kubectl create namespace network-policy-test
    ```
2.  **Create a test pod:**
    ```yaml title="nginx-pod.yaml"
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
      namespace: network-policy-test
    spec:
      containers:
        - name: nginx
          image: quay.io/bitnami/nginx:latest
          resources:
            requests:
              cpu: '100m'
              memory: '100Mi'
            limits:
              cpu: '200m'
              memory: '200Mi'
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
    ```
    ```bash
    kubectl apply -f nginx-pod.yaml
    ```
3.  **Create a default-deny network policy:**
    ```yaml title="default-deny.yaml"
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: default-deny
      namespace: network-policy-test
    spec:
      podSelector: {}
      policyTypes:
        - Ingress
    ```
    ```bash
    kubectl apply -f default-deny.yaml
    ```
4.  **Test connectivity:**
    ```bash
    kubectl run busybox --image=busybox --rm -it --restart=Never -n network-policy-test -- wget -T 5 <nginx-pod-ip>
    ```
    The `wget` command should time out, indicating that the default-deny policy is working.
5.  **Clean up:**
    ```bash
    kubectl delete namespace network-policy-test
    rm nginx-pod.yaml default-deny.yaml
    ```

## NPOL-002: Applying Baseline Network Policies

After verifying the Calico installation, the next step is to apply baseline network policies to your application namespaces. This starts with a "default deny" policy to ensure no traffic is allowed unless explicitly permitted.

### Default Deny (Ingress and Egress)

As per `NPOL-002`, we will create a policy that denies all ingress and egress traffic for all pods in the `frontend-dev` and `backend-dev` namespaces.

Create the file `infra/manifests/network-policies/frontend-dev/00-deny-all.yaml` and `infra/manifests/network-policies/backend-dev/00-deny-all.yaml` with the following content:

```yaml title="00-deny-all.yaml"
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

You can apply these policies by pointing to the directory:

```bash
kubectl apply -f infra/manifests/network-policies/frontend-dev/
kubectl apply -f infra/manifests/network-policies/backend-dev/
```

### Verifying Application Labels

Network policies rely on labels to select pods. It is crucial to ensure your application deployments have the correct labels that match the selectors in your policies.

During the verification process, it was discovered that the `backend-sample` deployment was missing the `app=backend` label, which is used in subsequent `NetworkPolicy` resources (e.g., `allow-from-frontend`).

You can check the labels of a deployment using the following command:

```bash
KUBECONFIG=$HOME/.kube/config kubectl get deployments -n backend-dev --show-labels
```

If a label is missing, you can add it using the `kubectl label` command:

```bash
KUBECONFIG=$HOME/.kube/config kubectl label deployment backend-sample -n backend-dev app=backend
```

After applying the label, the `backend-sample` deployment's labels will be updated, ensuring that network policies can correctly select it.

## Troubleshooting Application Deployments

This section details a real-world troubleshooting scenario that occurred while deploying the `frontend-sample` application.

### Problem: 504 Gateway Timeout

The `frontend-sample` application was returning a `504 Gateway Timeout` error when accessed via its URL (`https://dev.capitabyte.com`).

### Investigation

The investigation followed these steps:

1.  **Check Pod Status:** The `frontend-sample` pod was in a `Running` state, which was misleading.
2.  **Check Networking:** The `Ingress` and `Service` resources were correctly configured to route traffic to the pod on port 8000.
3.  **Test Internal Connectivity:** An attempt to `curl` the pod's IP address from within the cluster failed with a connection timeout. This confirmed that the application inside the pod was not responding.
4.  **Check Pod Logs:** The logs of the `frontend-sample` pod revealed the root cause:

    ```
    Error: listen EACCES: permission denied 0.0.0.0:80
    ```

    The application was trying to listen on port 80 instead of the intended port 8000.

5.  **Analyze Dockerfile and Deployment:**
    - The `Dockerfile` for the `frontend-sample` application correctly specified `CMD ["python", "-m", "http.server", "8000"]`.
    - The Kubernetes `Deployment` manifest was not overriding the `CMD`.
    - This led to the conclusion that the Docker image being used (`ghcr.io/bytequester/frontend-sample:328c382b827e97da1c1dae51ae65c77b0efa0f9d`) was stale and was built from an older version of the `Dockerfile` that used port 80.

### Solution: Patching the Deployment

The permanent solution is to rebuild the Docker image. However, an immediate fix was applied by patching the deployment to override the container's command and to satisfy the Kyverno security policies.

The initial patch attempt failed because it did not include the required `resources` and `securityContext` fields, which were being enforced by Kyverno.

The final, correct patch was:

```bash
KUBECONFIG=$HOME/.kube/config kubectl patch deployment frontend-sample -n frontend-dev --patch '{"spec":{"template":{"spec":{"containers":[{"name":"frontend-sample","command":["python","-m","http.server","8000"],"resources":{"requests":{"cpu":"100m","memory":"100Mi"},"limits":{"cpu":"200m","memory":"200Mi"}}}],"securityContext":{"runAsNonRoot":true,"runAsUser":1001}}}}}'
```

This patch did three things:

1.  Forced the container's `command` to use port 8000.
2.  Added `resources` with requests and limits to satisfy the `require-resource-limits` policy.
3.  Added a `securityContext` to run as a non-root user, satisfying the `require-run-as-non-root` policy.

After applying this patch, the `frontend-sample` deployment rolled out successfully, and the application became available.

## NPOL-003: Allow DNS Egress

After applying the default-deny policy, pods in the `frontend-dev` and `backend-dev` namespaces are not able to resolve DNS names. This policy allows pods to communicate with the CoreDNS service for DNS resolution.

### DNS Egress Policy

This policy allows egress traffic to the CoreDNS pods in the `kube-system` namespace on TCP and UDP port 53.

Create the file `infra/manifests/network-policies/frontend-dev/allow-dns.yaml` and `infra/manifests/network-policies/backend-dev/allow-dns.yaml` with the following content:

```yaml title="allow-dns.yaml"
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

### Verification

To verify that DNS is working, you can `exec` into a pod in the namespace and use `nslookup`. Since the application containers are minimal, a debug pod with networking tools is required.

1.  **Create a debug pod:**
    ```yaml title="debug-pod.yaml"
    apiVersion: v1
    kind: Pod
    metadata:
      name: tmp-debug
      namespace: frontend-dev
    spec:
      containers:
        - name: debug
          image: nicolaka/netshoot
          command: ['sleep', '3600']
          resources:
            requests:
              cpu: '100m'
              memory: '100Mi'
            limits:
              cpu: '200m'
              memory: '200Mi'
    ```
    ```bash
    kubectl apply -f debug-pod.yaml
    ```
2.  **Test DNS resolution:**

    ```bash
    KUBECONFIG=$HOME/.kube/config kubectl exec -n frontend-dev tmp-debug -- nslookup kubernetes.default
    ```

    A successful lookup confirms the policy is working.

3.  **Clean up:**
    ```bash
    kubectl delete pod tmp-debug -n frontend-dev
    rm debug-pod.yaml
    ```
