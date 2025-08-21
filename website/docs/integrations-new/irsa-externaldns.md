---
id: irsa-for-externaldns
slug: /integrations/irsa-for-externaldns
sidebar_position: 9
---

# IRSA for ExternalDNS

:::info To manage DNS records in AWS Route 53 automatically, the ExternalDNS controller needs AWS credentials. Instead of using static IAM user credentials, we use [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) to provide secure, fine-grained access. :::

## Overview

:::note IRSA allows us to associate an AWS IAM Role with a Kubernetes Service Account. Pods that use this service account are then able to assume the associated IAM role and are granted the permissions defined in that role's policy, without needing long-lived secrets stored in the cluster. :::

## 1. Creating the IAM Role with Terraform

We use a generic Terraform module located at `infra/modules/iam/irsa-generic` to create the necessary IAM resources. This module creates an IAM Role and attaches a policy that trusts the Kubernetes Service Account via the cluster's OIDC provider.

### Module Usage

Here is how you would use the module to create a role for ExternalDNS:

```terraform title="infra/modules/cluster/aws-eks/main.tf"
module "external_dns_irsa" {
  source = "../../iam/irsa-generic"

  role_name                 = "ExternalDNSRole"
  service_account_name      = "external-dns"
  service_account_namespace = "external-dns" # Assuming it's deployed here
  oidc_provider_arn         = module.eks.oidc_provider_arn # From the EKS module

  policy_json = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets"
      ],
      "Resource": "*"
    }
  ]
}
EOF
}
```

This Terraform code will output the ARN of the created role, which we need for the next step. For more details, see the module's README.

## 2. Configuring the Helm Chart

After the IAM role is created, we must configure the ExternalDNS Helm chart to use it. This is done by annotating the Kubernetes Service Account.

:::tip In your `values.yaml` for ExternalDNS (e.g., `infra/manifests/external-dns/values.yaml`), ensure the service account is created and annotated with the ARN of the role you created. :::

```yaml title="infra/manifests/external-dns/values.yaml"
provider: 'aws'
aws:
  region: 'eu-central-1'

rabc:
  create: true

serviceAccount:
  create: true
  name: 'external-dns'
  annotations:
    # This annotation is the key to linking the SA to the IAM Role
    eks.amazonaws.com/role-arn: 'arn:aws:iam::ACCOUNT_ID:role/ExternalDNSRole' # <-- Replace with the output from Terraform
```

## 3. Verification

Once you deploy the chart with these values, you can verify that IRSA is working correctly:

1.  **Check the Service Account annotations:**

    ```bash title="Check Service Account annotations"
    kubectl describe serviceaccount external-dns -n external-dns
    ```

    You should see the `eks.amazonaws.com/role-arn` annotation.

2.  **Check the pod's environment variables:** The EKS Pod Identity Webhook will automatically inject environment variables (`AWS_ROLE_ARN` and `AWS_WEB_IDENTITY_TOKEN_FILE`) into the `external-dns` pod.

    ```bash title="Check pod environment variables"
    kubectl describe pod -l app.kubernetes.io/name=external-dns -n external-dns
    ```

3.  **Check the logs:** The ExternalDNS logs should show that it is successfully finding and updating records in Route 53.
    ```bash title="Check ExternalDNS logs"
    kubectl logs -f -l app.kubernetes.io/name=external-dns -n external-dns
    ```
    Look for messages indicating successful connection to AWS and synchronization of records.
