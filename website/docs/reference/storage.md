---
id: storage
slug: /reference/storage
sidebar_position: 23
---

# Persistent Storage

:::info This document explains how to manage persistent storage in the cluster using StorageClasses and PersistentVolumeClaims (PVCs). :::

## 1. Check for a Default StorageClass

Most managed Kubernetes providers (GKE, EKS, AKS) come with a default `StorageClass` that is used to dynamically provision persistent volumes.

To see the available StorageClasses and identify the default, run:

```bash
kubectl get storageclass
```

The default StorageClass will have `(default)` next to its name.

```
NAME                 PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   kubernetes.io/gce-pd   Delete          Immediate           true                   2d
```

If no default StorageClass is available, you will need to create one.

## 2. Create a PersistentVolumeClaim

To request persistent storage, you create a `PersistentVolumeClaim` (PVC). If you don't specify a `storageClassName`, the default StorageClass will be used.

Here is an example of a PVC that requests 1Gi of storage:

```yaml title="pvc-example.yaml"
# pvc-example.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

To create this PVC, run:

```bash
kubectl apply -f pvc-example.yaml
```

## 3. Verify the PVC

To check the status of the PVC, run:

```bash
kubectl get pvc my-pvc
```

When the PVC is successfully bound to a PersistentVolume, its status will be `Bound`.

```
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-pvc    Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   1Gi        RWO            standard       30s
```

This PVC can now be mounted by a Pod to persist data.
