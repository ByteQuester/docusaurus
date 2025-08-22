---
id: backup-and-restore
slug: /runbooks/backup-and-restore
sidebar_position: 18
---

# Backup and Restore with Velero

:::info This document explains how to use Velero to back up and restore your Kubernetes cluster resources and persistent volumes. :::

## Prerequisites

Before you begin, you should have:

- `velero` CLI installed on your local machine.
- A running Kubernetes cluster.
- A storage provider (e.g., AWS S3, GCS) configured for your backups.

## 1. Install Velero

Follow the [Velero documentation](https://velero.io/docs/v1.10/basic-install/) to install Velero on your cluster. You will need to configure a storage provider to store your backups.

:::tip Configure a Storage Provider You will need to configure a storage provider for your backups (e.g., AWS S3, GCS). Refer to the Velero documentation for instructions on how to configure your specific provider. :::

## 2. Create a Backup

To create a one-time backup of the entire cluster, run the following command:

```bash title="Create a Velero backup"
velero backup create my-backup
```

You can also schedule regular backups. For example, to create a backup every 24 hours, run:

```bash title="Schedule a daily backup"
velero schedule create daily-backup --schedule="@every 24h"
```

## 3. Restore from a Backup

:::warning Potential Disruption Restoring from a backup can be a disruptive operation. It is recommended to perform restores in a non-production environment first. :::

To restore all resources from a backup, run the following command:

```bash title="Restore from a Velero backup"
velero restore create --from-backup my-backup
```

You can also restore specific resources from a backup. For example, to restore only the `nginx` deployment, run:

```bash title="Restore a specific resource"
velero restore create --from-backup my-backup --include-resources deployments/nginx
```

## 4. Verification

After a restore, you should verify that your resources have been restored correctly.

1.  **Check the restore status:**
    ```bash
    velero restore get
    ```
2.  **Inspect the restored resources:**
    ```bash
    kubectl get all -n <namespace>
    ```

## Best Practices

- **Regularly test your backups:** The only way to know if your backups are working is to test them.
- **Store your backups in a separate location:** Do not store your backups on the same cluster that you are backing up.
- **Encrypt your backups:** If your backups contain sensitive data, you should encrypt them.
