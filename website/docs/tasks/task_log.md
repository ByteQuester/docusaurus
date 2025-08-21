# Task Log: AWS Zero-to-Hero

This document provides a detailed log of the steps taken to complete the tasks in `00_aws_zero_to_hero.md`.

## AWSZ-001 — Prereqs and local toolchain

- **Status:** Completed
- **What we did:** We verified that all the necessary local tools (`kubectl`, `helm`, `aws`) were installed and configured correctly by running the acceptance criteria commands:
  - `kubectl version --client`
  - `helm version`
  - `aws sts get-caller-identity`

## AWSZ-002 — AWS account and IAM user/role for bootstrap

- **Status:** Completed
- **What we did:** We confirmed that the AWS credentials were correctly configured by verifying the output of `aws sts get-caller-identity`.

## AWSZ-003 — Terraform remote state backend (private)

- **Status:** Completed
- **What we did:** We set up a remote backend for Terraform to store its state in a shared and secure location.
  - Created an S3 bucket (`terraform-state-320819923517`) to store the Terraform state file.
  - Created a DynamoDB table (`terraform-state-locks`) for state locking to prevent concurrent modifications.
  - Configured the Terraform backend in `infra/modules/cluster/aws-eks/backend.tf` and added this file to `.gitignore`.
  - Documented the backend configuration in `infra/modules/cluster/aws-eks/README.md`.
- **Issues and Resolutions:**
  - **Issue:** The IAM user `mono` lacked permissions to create S3 buckets.
  - **Resolution:** The user manually attached the `AdministratorAccess` policy to the `mono` user in the AWS console.

## AWSZ-004 — EKS Terraform variables

- **Status:** Completed
- **What we did:** We created a `terraform.tfvars` file in `infra/modules/cluster/aws-eks` to provide values for the Terraform variables (`cluster_name` and `aws_region`).

## AWSZ-005 — Provision EKS cluster (Terraform)

- **Status:** Completed
- **What we did:** We provisioned the EKS cluster using `terraform apply`.
- **Issues and Resolutions:**
  - **Issue 1:** The EKS worker nodes failed to join the cluster because they were launched in private subnets with no route to the internet.
  - **Resolution 1:** We added an `aws_internet_gateway` and a route to `0.0.0.0/0` in the main route table by modifying `infra/modules/cluster/aws-eks/main.tf`.
  - **Issue 2:** The worker nodes were still failing because they were not being assigned public IP addresses.
  - **Resolution 2:** We enabled the "auto-assign public IP" setting for the subnets by adding `map_public_ip_on_launch = true` to the `aws_subnet.main` resource in `infra/modules/cluster/aws-eks/main.tf`.
  - **Issue 3:** The Terraform module did not create any worker nodes by default.
  - **Resolution 3:** We added resources to create an EKS node group in `infra/modules/cluster/aws-eks/main.tf`.

## AWSZ-006 — Kubectl context sanity checks

- **Status:** Completed
- **What we did:** We verified that `kubectl` was correctly configured to communicate with the new EKS cluster by running `kubectl config current-context` and `kubectl get pods -n kube-system`.

## AWSZ-007 — Ingress Controller (ingress-nginx)

- **Status:** Completed
- **What we did:** We installed the `ingress-nginx` controller using Helm to handle incoming traffic to our applications.

## AWSZ-008 — cert-manager (CRDs + controller)

- **Status:** Completed
- **What we did:** We installed `cert-manager` using Helm to automate TLS certificate management. We also configured it with `ClusterIssuer` resources for Let's Encrypt.
- **Code Changes:**
  - We used our generic IRSA Terraform module (`BP-005`) to create the IAM role for `cert-manager` DNS01 challenges (`BP-007`).

## AWSZ-009 — ExternalDNS

- **Status:** In Progress
- **What we did:** We installed `ExternalDNS` to automatically manage DNS records in Route 53.
- **Issues and Resolutions:**
  - **Issue:** `external-dns` was failing to assume the IAM role due to a complex series of issues with the IRSA setup (missing OIDC provider, incorrect role policy, conflicting authentication methods).
  - **Resolution:** We debugged the IRSA setup by creating the OIDC provider, creating the IAM role and policy using our generic Terraform module (`BP-006`), and configuring the `external-dns` Helm chart correctly.
- **Current Status:** `ExternalDNS` is now running correctly, but we are still in the process of fully verifying it.

## AWSZ-010 — Domain and DNS

- **Status:** Completed
- **What we did:** We created a hosted zone in Route 53 for the domain `capitabyte.com` and updated the name servers at the domain registrar.

## AWSZ-011 — CI policy files

- **Status:** Completed
- **What we did:** We created the security policy files `.gitleaks.toml`, `.trivyignore`, and `.checkov.yaml` in the `policy` directory to configure our CI security scanners.

## AWSZ-012 — Align Node version file

- **Status:** Completed
- **What we did:** We updated the `reusable-build-push.yml` CI workflow to use the Node.js version specified in the `.nvmrc` file.

## AWSZ-028 — Network Policies Baseline

- **Status:** Completed
- **What we did:** We applied a baseline set of network policies to the `frontend-dev` and `backend-dev` namespaces to enforce least-privilege traffic.
- **Rollback Plan:** To rollback the network policies, run the following commands:

```bash
kubectl delete -f infra/manifests/network-policies/frontend-dev/
kubectl delete -f infra/manifests/network-policies/backend-dev/
```

- **Smoke Test:** After deleting the policies, the following curl command should succeed:

```bash
kubectl exec -it tmp-debug -n frontend-dev --kubeconfig=$HOME/.kube/config -- curl -v http://backend-sample.backend-dev:80/api/health
```

-

## RBAC Baseline

- **Status:** In Progress
- **Pointer:** See `tasks/29_rbac_baseline.md` and subtasks under `tasks/rbac/` for inventory, alignment of chart ServiceAccounts, apply/verify steps, and rollback plan.

## Fresh Start Guide

- See `docs/docs/fresh-start-redeploy.md` for the single entry point to redeploy samples, validate, and reintroduce minimal policies.

## CAP-003 — Requests/limits audit

- Short note: Completed. See `tasks/CAP-003_requests-limits-audit.md` and `docs/docs/capacity-audit.md` for findings and follow-ups.
