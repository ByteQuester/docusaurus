---
id: backup-and-restore
slug: /runbooks/backup-and-restore
sidebar_position: 18
---

# Backup and Restore

:::info This document explains how to use Velero to back up and restore the cluster. :::

## 1. Install Velero

Follow the [Velero documentation](https://velero.io/docs/v1.10/basic-install/) to install Velero on your cluster.

:::tip Configure a Storage Provider You will need to configure a storage provider for your backups (e.g., AWS S3, GCS). :::

## 2. Create a Backup

To create a backup of the entire cluster, run the following command:

```bash title="Create a Velero backup"
velero backup create my-backup
```

## 3. Restore from a Backup

:::warning Potential Disruption Restoring from a backup can be a disruptive operation. It is recommended to perform restores in a non-production environment first. :::

To restore from a backup, run the following command:

```bash title="Restore from a Velero backup"
velero restore create --from-backup my-backup
```
