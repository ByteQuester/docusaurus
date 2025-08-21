---
id: scaling
slug: /reference/scaling
sidebar_position: 19
---

# Cluster Scaling

:::info This document outlines the strategy for scaling the EKS cluster using the Cluster Autoscaler. :::

## Node Autoscaler

To address the resource constraints in the cluster, we will implement a node autoscaler.

:::tip Cluster Autoscaler vs. Karpenter We have chosen the **Cluster Autoscaler** for its simplicity and quick setup. It will help us automatically scale the number of nodes in our EKS cluster based on the pending pod load.

While Karpenter offers more advanced features like bin-packing and dynamic instance types, the Cluster Autoscaler is the more appropriate choice for now to quickly resolve the recurring "Too many pods" issue. We can consider migrating to Karpenter in the future if our needs become more complex. :::

<details>
  <summary>Cluster Autoscaler Installation</summary>

We will use the official Helm chart to install the Cluster Autoscaler.

1.  **Add the Helm repository:**

    ```bash
    helm repo add autoscaler https://kubernetes.github.io/autoscaler
    helm repo update
    ```

2.  **Create a `values.yaml` file:**

    Create a file named `infra/manifests/cluster-autoscaler/values.yaml` with the following content:

    ```yaml title="infra/manifests/cluster-autoscaler/values.yaml"
    autoDiscovery:
      clusterName: hosting-eks-cluster

    awsRegion: eu-central-1

    extraArgs:
      balance-similar-node-groups: true
      skip-nodes-with-local-storage: false
      expander: least-waste

    rbac:
      serviceAccount:
        create: true
        name: cluster-autoscaler
        annotations:
          eks.amazonaws.com/role-arn: arn:aws:iam::320819923517:role/AmazonEKSClusterAutoscalerRole
    ```

3.  **Create the IAM Role for Service Account (IRSA):**

    The Cluster Autoscaler needs IAM permissions to interact with AWS Auto Scaling Groups. We will use IRSA to associate an IAM role with the Cluster Autoscaler's service account.

    We will use the AWS CLI to create the necessary IAM role and policy.

    - **Create the Trust Policy JSON:**

      Create a file named `trust-policy.json` with the following content:

      ```json title="trust-policy.json"
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "Federated": "arn:aws:iam::320819923517:oidc-provider/oidc.eks.eu-central-1.amazonaws.com/id/D2227B82B08C7EA7410F6D956FE3FAFD"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
              "StringEquals": {
                "oidc.eks.eu-central-1.amazonaws.com/id/D2227B82B08C7EA7410F6D956FE3FAFD:sub": "system:serviceaccount:kube-system:cluster-autoscaler"
              }
            }
          }
        ]
      }
      ```

    - **Create the IAM Policy:**

      ```bash
      aws iam create-policy --policy-name AmazonEKSClusterAutoscalerPolicy --policy-document '{
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Action": [
                      "autoscaling:DescribeAutoScalingGroups",
                      "autoscaling:DescribeAutoScalingInstances",
                      "autoscaling:DescribeLaunchConfigurations",
                      "autoscaling:DescribeTags",
                      "autoscaling:SetDesiredCapacity",
                      "autoscaling:TerminateInstanceInAutoScalingGroup",
                      "ec2:DescribeLaunchTemplateVersions"
                  ],
                  "Resource": "*",
                  "Effect": "Allow"
              }
          ]
      }'
      ```

    - **Create the IAM Role:**

      ```bash
      aws iam create-role --role-name AmazonEKSClusterAutoscalerRole --assume-role-policy-document file://trust-policy.json
      ```

    - **Attach the IAM Policy to the Role:**

      ```bash
      aws iam attach-role-policy --role-name AmazonEKSClusterAutoscalerRole --policy-arn arn:aws:iam::320819923517:policy/AmazonEKSClusterAutoscalerPolicy
      ```

4.  **Install the Helm chart:**

    ```bash
    helm upgrade --install cluster-autoscaler autoscaler/cluster-autoscaler -n kube-system -f infra/manifests/cluster-autoscaler/values.yaml
    ```

5.  **Verify the installation:**

          Check the logs of the Cluster Autoscaler pod to ensure it starts without any credential errors:

          ```bash
          kubectl --namespace=kube-system logs -l "app.kubernetes.io/name=aws-cluster-autoscaler,app.kubernetes.io/instance=cluster-autoscaler"
          ```

    </details>
