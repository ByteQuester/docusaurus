---
id: cost-controls
slug: /reference/cost-controls
sidebar_position: 6
---

# Risk and Cost Controls

:::info This document provides guidance on managing risks and controlling costs for this project. :::

## Instance Sizing

Choosing the right instance size for our Kubernetes cluster nodes is crucial for both performance and cost-effectiveness.

- **Development Environment:** For the `dev` environment, we can use smaller, more cost-effective instances (e.g., `t3.medium` on AWS, `e2-medium` on GCP).
- **Production Environment:** For the `prod` environment, we should use instances with more resources to handle production traffic (e.g., `m5.large` on AWS, `n2-standard-2` on GCP).

:::tip We should monitor our resource utilization and adjust the instance sizes as needed to optimize for cost and performance. :::

## Autoscaling

We use Horizontal Pod Autoscaling (HPA) to automatically scale the number of application pods based on CPU and memory utilization. This helps us to handle traffic spikes without over-provisioning resources.

The HPA configuration can be found in the `values.yaml` file for our Helm chart and is enabled in the `values-prod.yaml` file.

We also use Cluster Autoscaler to automatically adjust the number of nodes in our Kubernetes cluster based on the resource demands of the pods.

## Shutdown Procedures

To save costs, we should shut down non-production environments when they are not in use.

- **Development Environment:** The `dev` environment can be scaled down to zero nodes during off-hours and weekends.
- **Staging/QA Environments:** Any staging or QA environments should also be scaled down when not in use.

We can use tools like [Kube-downscaler](https://github.com/kube-downscaler/kube-downscaler) or custom scripts to automate the shutdown and startup of our non-production environments.
