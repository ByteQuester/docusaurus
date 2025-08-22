---
id: dns-troubleshooting-runbook
slug: /runbooks/dns-troubleshooting
sidebar_position: 1
---

# DNS Troubleshooting Runbook

This runbook provides a step-by-step guide to troubleshoot and resolve DNS issues in the EKS cluster.

## 1. Triage

### 1.1. Control Plane Connectivity

**Goal:** Confirm that nodes/pods can reach the Kubernetes API server.

**Commands:**

```bash
# Check kube-proxy and coredns pods
kubectl get pods -n kube-system -l k8s-app=kube-proxy,k8s-app=kube-dns -o wide

# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy --tail=200

# Test API server connectivity from a debug pod
kubectl apply -f ops/debug/debug-pod-kube-system.yaml
kubectl exec -n kube-system debug-kube-system -- sh -c "wget -qO- https://kubernetes.default.svc"
```

### 1.2. CoreDNS Health and Config

**Goal:** Check the health and configuration of CoreDNS.

**Commands:**

```bash
# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide

# Describe CoreDNS pods
kubectl describe pod -n kube-system -l k8s-app=kube-dns

# Check CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=200

# Check CoreDNS service
kubectl get svc -n kube-system kube-dns -o wide

# Check CoreDNS configmap
kubectl get cm -n kube-system coredns -o yaml
```

### 1.3. CNI and NetworkPolicy Engine Alignment

**Goal:** Ensure the CNI and NetworkPolicy engine are correctly configured.

**Commands:**

```bash
# Check Calico components
kubectl get pods -n tigera-operator,calico-system

# Check network policies
kubectl get netpol -A
```

## 2. Remediation

### 2.1. EKS Endpoint and Security Groups Validation

**Goal:** Validate that worker node security groups allow egress to the EKS API endpoint.

**Steps:**

1.  Identify EKS API endpoint IPs:
    ```bash
    aws eks describe-cluster --name <cluster_name> --query 'cluster.endpoint' --output text
    nslookup <endpoint-host>
    ```
2.  Verify worker node security groups allow egress to those IPs on TCP/443.
3.  Confirm route tables/NACLs allow return traffic.

### 2.2. kube-proxy and ClusterIP Path Validation

**Goal:** Verify kube-proxy and ClusterIP routing.

**Commands:**

```bash
# Check kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy --tail=200

# Test ClusterIP routing from a debug pod
kubectl exec -n frontend-dev tmp-debug -- getent hosts backend-sample.backend-dev.svc.cluster.local
kubectl exec -n frontend-dev tmp-debug -- sh -c 'IP=$(getent hosts backend-sample.backend-dev | awk "{print \$1}"); [ -n "$IP" ] && wget -qO- http://$IP:80/api/health'
```

### 2.3. Cleanup Stray Files and Temporary Policies

**Goal:** Normalize ad-hoc YAMLs and remove temporary policies.

**Action:** Review any temporary files created during the incident and either remove them or move them to the appropriate location.

## 3. Validation

### 3.1. End-to-End DNS and Site Validation

**Goal:** Validate DNS resolution and site functionality.

**Commands:**

```bash
# Internal DNS check
kubectl exec -n frontend-dev tmp-debug -- nslookup kubernetes.default.svc
kubectl exec -n frontend-dev tmp-debug -- nslookup backend-sample.backend-dev.svc

# App connectivity
kubectl exec -n frontend-dev tmp-debug -- curl -sv http://backend-sample.backend-dev:80/api/health

# External connectivity
curl -sv https://dev.capitabyte.com/api/health
curl -sv https://dev.capitabyte.com/
```

## 4. Post-Incident Actions

### 4.1. Update Documentation

**Goal:** Update relevant documentation with the findings from the incident.

**Action:**

- Create a post-incident report and store it in `runbooks-new/incidents/`.
- Update this runbook with any new findings or steps.
