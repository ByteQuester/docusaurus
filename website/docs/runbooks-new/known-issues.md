---
id: known-issues
slug: /runbooks/known-issues
sidebar_position: 8
---

# Known Issues

:::info This document lists known issues and workarounds for this project. :::

## Calico Installation Blocked by Kyverno

:::warning Issue When installing Calico on a cluster with Kyverno policies, the installation may be blocked by the `require-image-registry`, `require-resource-limits`, and `require-run-as-non-root` policies. :::

:::tip Workaround Temporarily set the `validationFailureAction` to `audit` for these policies during the Calico installation. After the installation is complete, you can revert the policies to `enforce` mode. :::

## Metrics Visibility Gotchas

:::warning Issue Prometheus may not be able to scrape metrics from all components by default. :::

:::tip Workaround You may need to create a `ServiceMonitor` or `PodMonitor` resource to enable Prometheus to scrape metrics from a specific component. Refer to the Prometheus documentation for more information. :::
