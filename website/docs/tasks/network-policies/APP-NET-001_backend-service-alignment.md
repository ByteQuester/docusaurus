# APP-NET-001 — Validate backend-sample Service and Deployment alignment

- Status: Completed
- Dependencies: NPOL-TRIAGE-003

## Steps

We reviewed the `backend-sample` Service and Deployment to ensure they were aligned.

**Service:**

```bash
kubectl describe svc -n backend-dev backend-sample --kubeconfig=$HOME/.kube/config
```

**Output:**

```
Name:                     backend-sample
Namespace:                backend-dev
Labels:                   app.kubernetes.io/instance=backend-sample
                          app.kubernetes.io/managed-by=Helm
                          app.kubernetes.io/name=backend-sample
                          app.kubernetes.io/version=1.0
                          helm.sh/chart=backend-sample-0.1.0
Annotations:              argocd.argoproj.io/tracking-id: backend-sample:/Service:backend-dev/backend-sample
                          meta.helm.sh/release-name: backend-sample
                          meta.helm.sh/release-namespace: backend-dev
Selector:                 app.kubernetes.io/instance=backend-sample,app.kubernetes.io/name=backend-sample
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       172.20.170.224
IPs:                      172.20.170.224
Port:                     http  80/TCP
TargetPort:               http/TCP
Endpoints:                10.0.0.85:3001,10.0.1.142:3001
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

**Deployment:**

```bash
kubectl describe deploy -n backend-dev backend-sample --kubeconfig=$HOME/.kube/config
```

**Output (partial):**

```
Containers:
   backend-sample:
    Image:      ghcr.io/bytequester/backend-sample:7564dfbe52b00c57ef3ce691323987365613a592
    Port:       3001/TCP
    Host Port:  0/TCP
```

## Analysis

The Service `targetPort` is `http`, which correctly maps to the container's port named `http` on port `3001`. The Service selectors also match the Deployment's labels, and the Endpoints are correctly populated.

## Acceptance

- Service selectors match Deployment labels; endpoints populated for backend pod port.
