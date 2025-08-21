---
id: cluster-provisioning
slug: /guides/cluster-provisioning
sidebar_position: 5
---

# Cluster Provisioning

:::info This guide provides instructions to provision a Kubernetes cluster on either Google Cloud Platform (GCP) or Amazon Web Services (AWS) using the provided Terraform modules. :::

## Prerequisites

:::tip Before you begin Ensure you have the following tools installed and configured:

- **Terraform**: [Installation Guide](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- **kubectl**: [Installation Guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- **Cloud Provider CLI**:
  - For GCP: **gcloud CLI** - [Installation Guide](https://cloud.google.com/sdk/docs/install)
  - For AWS: **AWS CLI** - [Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

You must also be authenticated with your chosen cloud provider. :::

## 1. Configure Remote State Backend

To securely store your Terraform state, you must configure a remote backend. This is a critical step to ensure state is not stored in the local repository.

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem';

<Tabs>
  <TabItem value="gcp" label="GCP (Google Cloud Storage)">

1.  **Create a GCS Bucket**:

    ```sh
    gsutil mb gs://<your-unique-bucket-name>/
    ```

2.  **Enable Versioning**:

    ```sh
    gsutil versioning set on gs://<your-unique-bucket-name>
    ```

3.  Create a file named `backend.tf` in the appropriate module directory (`infra/modules/cluster/gcp-gke`):

    ```terraform title="infra/modules/cluster/gcp-gke/backend.tf"
    terraform {
      backend "gcs" {
        bucket  = "<your-unique-bucket-name>"
        prefix  = "terraform/state"
      }
    }
    ```

    :::danger Do Not Commit `backend.tf` This file contains sensitive information and should **not** be committed to the repository. Add `backend.tf` to your global `.gitignore` file. ::: </TabItem> <TabItem value="aws" label="AWS (Amazon S3)">

4.  **Create an S3 Bucket**:

    ```sh
    aws s3api create-bucket --bucket <your-unique-bucket-name> --region <your-region>
    ```

5.  Create a file named `backend.tf` in `infra/modules/cluster/aws-eks`:

        ```terraform title="infra/modules/cluster/aws-eks/backend.tf"
        terraform {
          backend "s3" {
            bucket  = "<your-unique-bucket-name>"
            key     = "eks/terraform.tfstate"
            region  = "<your-region>"
          }
        }
        ```
        :::danger Do Not Commit `backend.tf`
        This file contains sensitive information and should **not** be committed to the repository. Add `backend.tf` to your global `.gitignore` file.
        :::

      </TabItem>
    </Tabs>

## 2. Provision the Cluster

Navigate to the directory of your chosen provider to run the provisioning commands.

<Tabs>
  <TabItem value="gcp" label="GCP (GKE)">

1.  **Navigate to the module**:

    ```sh
    cd infra/modules/cluster/gcp-gke
    ```

2.  **Initialize Terraform**: This will initialize the backend and download the necessary providers.

    ```sh
    terraform init
    ```

3.  **Review and Apply**: Create a `terraform.tfvars` file to specify your project and region, or pass them as variables.

    ```sh
    terraform apply
    ```

    Review the plan and type `yes` to proceed. </TabItem> <TabItem value="aws" label="AWS (EKS)">

4.  **Navigate to the module**:

    ```sh
    cd infra/modules/cluster/aws-eks
    ```

5.  **Initialize Terraform**:

    ```sh
    terraform init
    ```

6.  **Review and Apply**: Create a `terraform.tfvars` file to specify your cluster name and other variables. `sh terraform apply ` Review the plan and type `yes` to proceed. </TabItem> </Tabs>

## 3. Verify Cluster Access

Once provisioning is complete, Terraform will output the necessary information to configure `kubectl`.

1.  **Configure `kubectl`**:

    - For **GKE**, use the `gcloud` command provided in the Terraform output.
    - For **EKS**, use the `aws` command provided in the Terraform output.

2.  **Check Node Status**: Run the following command to verify that your nodes are ready:

    ```sh
    kubectl get nodes
    ```

    You should see a list of nodes with the status `Ready`.

    ```
    NAME                                     STATUS   ROLES    AGE   VERSION
    gke-my-cluster-default-pool-xxxx-yyyy   Ready    <none>   2m    v1.28.x-gke.xxxx
    gke-my-cluster-default-pool-xxxx-zzzz   Ready    <none>   2m    v1.28.x-gke.xxxx
    ```

:::success Congratulations! You have now successfully provisioned a Kubernetes cluster. :::
