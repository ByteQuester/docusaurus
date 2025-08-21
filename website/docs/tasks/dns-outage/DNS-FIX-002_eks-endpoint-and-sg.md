# DNS-FIX-002 — EKS endpoint and security groups validation

- Status: In Progress
- Dependencies: DNS-TRIAGE-001

## Scope

Validate that worker node security groups allow egress to the EKS API endpoint and that route tables/NACLs are not blocking.

## Steps

1. Identify EKS API endpoint IPs:

```bash
aws eks describe-cluster --name <cluster_name> --query 'cluster.endpoint' --output text
nslookup <endpoint-host> || dig +short <endpoint-host>
```

2. Verify worker node security groups allow egress to those IPs on TCP/443.

3. Confirm route tables/NACLs allow return traffic.

## Acceptance

- Security groups and networking allow nodes/pods to reach the API server on 443.
