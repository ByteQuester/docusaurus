---
id: debugging-report-dns
slug: /runbooks/incidents/debugging-report-dns
sidebar_position: 1
---

# Debugging Report: Website Connectivity Issue

:::info Summary The website is down and returning an `ERR_CONNECTION_CLOSED` error. After a thorough investigation, the root cause has been identified as a failure of the CoreDNS pods to connect to the Kubernetes API server. This is preventing DNS resolution from working correctly within the cluster, which in turn is causing the website to be unreachable. :::

## Initial Problem

The initial problem was that the application was unreachable with an `ERR_CONNECTION_CLOSED` error. The ingress controller logs showed "upstream timed out" errors when trying to connect to the application pods.

## Debugging Steps

1.  **Checked Pods and Services:**

    - Verified that the `frontend-sample` and `backend-sample` pods were running and their logs were clean.
    - Identified that the `frontend` service had no endpoints.

2.  **Corrected Service Selector:**

    - Found that the `frontend` service selector did not match the pod labels.
    - Corrected the service selector, which resulted in the service getting the correct endpoints.

3.  **Verified Network Policies:**

    - Reviewed the network policies in the `frontend-dev` and `ingress` namespaces.
    - Created a temporary network policy to allow traffic from a debug pod in the `default` namespace to the `frontend-dev` namespace.

4.  **Investigated DNS Resolution:**

    - Found that DNS resolution was not working from the debug pod.
    - Verified that the CoreDNS pods were running and the `kube-dns` service was correctly configured.

5.  **Enabled Network Policies in CNI:**

    - Discovered that network policies were not enabled in the AWS VPC CNI.
    - Enabled network policies in the `aws-node` daemonset and restarted the `aws-node` pods.

6.  **Identified CoreDNS to API Server Connectivity Issue:**

    - Found that the CoreDNS pods were not in a `Ready` state because their readiness probes were failing.
    - The CoreDNS logs showed that they were unable to connect to the Kubernetes API server with a "dial tcp 172.20.0.1:443: i/o timeout" error.

7.  **Verified API Server Connectivity:**
    - Created a debug pod in the `kube-system` namespace and tried to curl the API server, but the connection timed out.

## Root Cause Analysis

:::warning Root Cause The root cause of the problem is that the CoreDNS pods are unable to connect to the Kubernetes API server. This is preventing DNS resolution from working correctly within the cluster. The exact reason for the lack of connectivity is still unknown, but it is likely a networking issue within the EKS cluster, such as a security group misconfiguration, a routing issue, or a problem with the AWS VPC CNI itself. :::

## Current Status

:::note

- The website is down.
- DNS resolution is not working in the cluster.
- The CoreDNS pods are not in a `Ready` state. :::

## Next Steps

:::danger Escalation Required I have exhausted my debugging options. I recommend escalating this issue to a network or EKS expert to investigate the networking configuration of the cluster. :::
