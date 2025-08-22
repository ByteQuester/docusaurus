---
id: tls-troubleshooting
slug: /runbooks/tls-troubleshooting
sidebar_position: 21
---

# TLS Troubleshooting

:::info This document contains the output of the troubleshooting steps performed to recover the cert-manager installation. :::

## Problem Statement

The cert-manager installation was in a failed state. The `helm list` command showed the `cert-manager` release as `failed`, and the necessary resources (pods, services, etc.) were not created in the `cert-manager` namespace.

## Investigation Diary

### CERT-001 — Snapshot current state

#### `helm list -A | grep cert-manager`

```
cert-manager   	cert-manager	1       	2025-08-16 10:55:58.203530783 +0000 UTC	failed  	cert-manager-v1.14.5 	v1.14.5
```

#### `sudo kubectl get ns cert-manager -o yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: '2025-08-16T10:47:29Z'
  labels:
    kubernetes.io/metadata.name: cert-manager
  name: cert-manager
  resourceVersion: '41687'
  uid: b7fc2c46-fb23-4858-a9f1-963d1de8fac6
spec:
  finalizers:
    - kubernetes
status:
  phase: Active
```

#### `sudo kubectl get deploy,sts,ds,po,svc,sa,role,rolebinding -n cert-manager`

```
NAME                     SECRETS   AGE
serviceaccount/default   0         16m
```

#### `sudo kubectl get crd | grep cert-manager.io`

```
certificaterequests.cert-manager.io         2025-08-16T10:54:57Z
certificates.cert-manager.io                2025-08-16T10:54:57Z
challenges.acme.cert-manager.io             2025-08-16T10:54:57Z
clusterissuers.cert-manager.io              2025-08-16T10:54:57Z
issuers.cert-manager.io                     2025-08-16T10:54:57Z
orders.acme.cert-manager.io                 2025-08-16T10:54:57Z
```

#### `sudo kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations | grep cert-manager`

(No output)

#### `sudo kubectl get events -A | grep -i cert-manager`

(No output)

## Root Cause

The core issue was an inconsistent Kubernetes context between `helm` and `kubectl` commands, further complicated by a Helm chart installation that repeatedly failed to create the necessary resources (pods and CRDs) despite reporting success.

## Resolution

The following steps were taken to resolve the issue:

1.  **Align Kube Context:** Stopped using `sudo` for `kubectl` and consistently used the `--kubeconfig $HOME/.kube/config` flag for all `helm` and `kubectl` commands.
2.  **Clean Uninstall:** Completely purged the failed `cert-manager` installation, including the Helm release, secrets, CRDs, and webhook configurations.
3.  **Manual CRD Installation:** Installed the `cert-manager` CRDs manually from the official YAML file for version `v1.14.5`.
4.  **Helm Install without CRDs:** Installed the `cert-manager` Helm chart (version `v1.14.5`) without the `crds.enabled` flag, as the CRDs were already installed.
5.  **Fix Ingress Template:** The `ingress.yaml` template in the `frontend-sample` chart was missing the `tls` section. This was added to the template.
6.  **Issue Certificates:** After the fixes, the staging and production certificates were successfully issued.

:::success Final Working Chart Version

- **cert-manager:** `v1.14.5` :::
