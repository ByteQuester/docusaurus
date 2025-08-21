# NPOL-004 — Allow frontend → backend service traffic

- Status: Completed
- Dependencies: NPOL-002

## Manifests

- `infra/manifests/network-policies/backend-dev/10-allow-from-frontend.yaml`

## Verification

We first checked for the `10-allow-from-frontend` network policy in the `backend-dev` namespace and found it was not present. We applied it from the repository.

```bash
kubectl apply -f infra/manifests/network-policies/backend-dev/10-allow-from-frontend.yaml --kubeconfig=$HOME/.kube/config
```

We then verified that the policy was applied successfully.

```bash
kubectl get netpol -n backend-dev --kubeconfig=$HOME/.kube/config
```

**Output:**

```
NAME                  POD-SELECTOR                            AGE
allow-dns             <none>                                  101s
allow-from-frontend   app.kubernetes.io/name=backend-sample   4s
default-deny-all      <none>                                  4m25s
```

We then attempted to verify connectivity from the frontend pod to the backend service. This led to a lengthy troubleshooting process.

### Troubleshooting

1.  **Initial check from `frontend-sample` pod:** The initial attempt to `curl` the backend from the `frontend-sample` pod failed because `curl` was not available in the container.

2.  **Debug pod:** We created a debug pod (`tmp-debug`) in the `frontend-dev` namespace using the `nicolaka/netshoot` image, which contains a rich set of networking tools.

3.  **Internal check from debug pod:** The `curl` from the debug pod to the backend service timed out. This indicated an egress issue from the `frontend-dev` namespace.

4.  **Egress policy investigation:** We discovered that while there was a `default-deny-all` policy on the `frontend-dev` namespace, there was no corresponding egress policy to allow traffic to the backend.

5.  **Trial and error with egress policies:** We experimented with several egress policies:

    - An egress policy to the backend pods on port 80 (failed).
    - An egress policy to the backend pods on port 3001 (failed).
    - An egress policy to the backend namespace on port 80 (failed).
    - An egress policy that allowed all egress traffic (succeeded).
    - An egress policy that allowed all egress traffic to the `backend-dev` namespace (succeeded).

6.  **Final working policy:** The policy that finally worked was one that allowed all egress traffic from the `frontend-dev` namespace to the `backend-dev` namespace. This was created in the file `infra/manifests/network-policies/frontend-dev/allow-to-backend-dev-namespace.yaml`.

### Final Verification

With the correct egress policy in place, the internal check from the debug pod was successful.

```bash
kubectl exec tmp-debug -n frontend-dev --kubeconfig=$HOME/.kube/config -- curl -sv http://backend-sample.backend-dev:80/api/health
```

**Output:**

```
* Host backend-sample.backend-dev:80 was resolved.
* IPv6: (none)
* IPv4: 172.20.170.224
*   Trying 172.20.170.224:80...
* Connected to backend-sample.backend-dev (172.20.170.224) port 80
* using HTTP/1.x
> GET /api/health HTTP/1.1
> Host: backend-sample.backend-dev
> User-Agent: curl/8.14.1
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 200 OK
< X-Powered-By: Express
< Content-Type: text/html; charset=utf-8
< Content-Length: 2
< ETag: W/"2-nOO9QiTIwXgNtWtBJezz8kv3SLc"
< Date: Wed, 20 Aug 2025 07:56:37 GMT
< Connection: keep-alive
< Keep-Alive: timeout=5
<
{ [2 bytes data]
* Connection #0 to host backend-sample.backend-dev left intact
OK
```

## Acceptance

- Internal request from frontend namespace to backend Service succeeds (HTTP 200).
