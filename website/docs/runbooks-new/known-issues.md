---
id: known-issues
slug: /runbooks/known-issues
sidebar_position: 8
---

# Known Issues

:::info This document lists known issues and workarounds for this project. If you encounter an issue that is not listed here, please report it. :::

## Calico Installation Blocked by Kyverno

**Problem:** When installing Calico on a cluster with Kyverno policies, the installation may be blocked.

**Cause:** The `require-image-registry`, `require-resource-limits`, and `require-run-as-non-root` Kyverno policies can prevent the Calico pods from being created.

**Solution:** Temporarily set the `validationFailureAction` to `audit` for these policies during the Calico installation. After the installation is complete, you can revert the policies to `enforce` mode.

## Metrics Visibility Gotchas

**Problem:** Prometheus may not be able to scrape metrics from all components by default.

**Cause:** Prometheus requires a `ServiceMonitor` or `PodMonitor` resource to be created to scrape metrics from a specific component.

**Solution:** You may need to create a `ServiceMonitor` or `PodMonitor` resource to enable Prometheus to scrape metrics from a specific component. Refer to the Prometheus documentation for more information.

## Reporting New Issues

If you encounter an issue that is not listed here, please report it by creating a new issue in the [GitHub repository](https://github.com/ByteQuester/hosting-monorepo/issues).
