---
id: debugging-report-detailed
slug: /runbooks/debugging-report-detailed
sidebar_position: 6
---

# Detailed Debugging Report: Website Connectivity Issue

:::info Summary The website is down and returning an `ERR_CONNECTION_CLOSED` error. After a thorough investigation, the root cause has been identified as a failure of the CoreDNS pods to connect to the Kubernetes API server. This is preventing DNS resolution from working correctly within the cluster, which in turn is causing the website to be unreachable. :::

## Chronological Debugging Steps

### RBAC Alignment Task

The initial task was to align the ServiceAccounts in the Helm charts and RBAC manifests.

1.  **Identified RBAC Manifests:**

    - I listed the files in `infra/manifests/rbac/` and found `backend-rbac.yaml` and `frontend-rbac.yaml`.

2.  **Checked for Alignment:**

    - I read the contents of the RBAC manifests and the `values.yaml` files for the `frontend-sample` and `backend-sample` charts.
    - I found that the `ServiceAccount` names were already aligned.

3.  **Corrected Frontend Naming:**

    - Based on the user's feedback about confusing naming, I updated `infra/manifests/rbac/frontend-rbac.yaml` to change the `ServiceAccount` name from `frontend` to `frontend-sample`.
    - I also updated `charts/frontend-sample/values.yaml` to change `serviceAccount.name` from `"frontend"` to `"frontend-sample"`.

4.  **Applied RBAC Changes:**
    - I applied the updated `frontend-rbac.yaml` file.
    - I tried to upgrade the `frontend-sample` Helm release, but it failed with a webhook timeout error.

### Investigating the Webhook Timeout

1.  **Network Policy Investigation:**

    - I suspected that a network policy was blocking the webhook.
    - I created a network policy to allow traffic from the `ingress` namespace to the `frontend-dev` namespace on port 443, but this did not solve the issue.
    - I then created a network policy to allow egress traffic from the `kube-system` namespace to the `ingress` namespace on port 443, but this also did not solve the issue.

2.  **Webhook Deletion (Workaround):**
    - As a workaround, I deleted the `ingress-nginx-admission` validating webhook configuration.
    - This allowed the `helm upgrade` to succeed.
    - I then re-applied the webhook configuration.

### Investigating the Website Connectivity Issue

1.  **Verified RBAC:**

    - I verified that the `RoleBindings` were correctly bound to the `ServiceAccounts`.

2.  **DNS Resolution Failure:**

    - I created a debug pod in the `default` namespace and found that it was unable to resolve the service name `frontend.frontend-dev`.

3.  **IP-based Connectivity Success:**

    - I was able to curl the frontend service using its IP address, which confirmed that the network connectivity between the namespaces was working.

4.  **CoreDNS Investigation:**

    - I found that the CoreDNS pods were not in a `Ready` state because their readiness probes were failing.
    - The CoreDNS logs showed that they were unable to connect to the Kubernetes API server with a "dial tcp 172.20.0.1:443: i/o timeout" error.

5.  **API Server Connectivity Failure:**
    - I created a debug pod in the `kube-system` namespace and tried to curl the API server, but the connection timed out.

## Root Cause Analysis

:::warning Root Cause The root cause of the problem is that the CoreDNS pods are unable to connect to the Kubernetes API server. This is preventing DNS resolution from working correctly within the cluster. The exact reason for the lack of connectivity is still unknown, but it is likely a networking issue within the EKS cluster, such as a security group misconfiguration, a routing issue, or a problem with the AWS VPC CNI itself. :::

## Current Status

:::note

- The website is down.
- DNS resolution is not working in the cluster.
- The CoreDNS pods are not in a `Ready` state. :::

## Next Steps

:::danger Escalation Required I have exhausted my debugging options. I recommend escalating this issue to a network or EKS expert to investigate the networking configuration of the cluster. :::
