---
id: pod-security-admission
slug: /reference/pod-security-admission
sidebar_position: 16
---

# Pod Security Admission

:::info This document explains how to use Pod Security Admission (PSA) to enforce pod security standards in your cluster. :::

## What is Pod Security Admission?

Pod Security Admission is a built-in Kubernetes admission controller that implements the security standards defined in the [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/).

There are three PSA levels:

- **privileged**: Unrestricted.
- **baseline**: Minimally restrictive, while preventing known privilege escalations.
- **restricted**: Heavily restricted, following current pod hardening best practices.

## Applying PSA to a Namespace

PSA is configured by applying labels to namespaces. We will enforce the `restricted` profile on our application namespaces.

To apply the `restricted` profile to the `frontend-dev` namespace, run:

```bash title="Apply restricted PSA to frontend-dev namespace"
kubectl label --overwrite ns frontend-dev pod-security.kubernetes.io/enforce=restricted
```

This will deny any new pods in the `frontend-dev` namespace that do not meet the `restricted` policy.

## Example of a Denied Pod

The following pod would be denied by the `restricted` policy because it runs as root and does not define security capabilities.

```yaml title="pod-denied-example.yaml"
# pod-denied-example.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-denied-pod
spec:
  containers:
    - name: web
      image: nginx
```

To be admitted, the pod would need a `securityContext` that meets the `restricted` policy requirements.
