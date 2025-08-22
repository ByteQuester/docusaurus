---
id: npol-validate-001-functional-checks
slug: /archive/network-policies-tasks/npol-validate-001-functional-checks
sidebar_position: 18
---

# NPOL-VALIDATE-001 — Functional checks under policies

- **Status**: Completed
- **Dependencies**: NPOL-TRIAGE-002..003, APP-NET-002, GITOPS-ALIGN-001

## Internal check

After creating a debug pod, we were able to perform the internal check.

```bash title="Internal connectivity check"
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

## External check

The external check was performed as part of `NPOL-005` and was successful.

```bash title="External connectivity check"
curl -sv https://dev.capitabyte.com/api/health
```

**Output:**

```
*   Trying 3.72.240.252:443...
* Connected to dev.capitabyte.com (3.72.240.252) port 443 (#0)
* ALPN: offers h2,http/1.1
} [5 bytes data]
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
} [512 bytes data]
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: /etc/ssl/certs
{ [5 bytes data]
* TLSv1.3 (IN), TLS handshake, Server hello (2):
{ [122 bytes data]
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
{ [41 bytes data]
* TLSv1.3 (IN), TLS handshake, Certificate (11):
{ [2591 bytes data]
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
{ [264 bytes data]
* TLSv1.3 (IN), TLS handshake, Finished (20):
{ [52 bytes data]
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
} [1 bytes data]
* TLSv1.3 (OUT), TLS handshake, Finished (20):
} [52 bytes data]
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=dev.capitabyte.com
*  start date: Aug 16 15:44:44 2025 GMT
*  expire date: Nov 14 15:44:43 2025 GMT
*  subjectAltName: host "dev.capitabyte.com" matched cert's "dev.capitabyte.com"
*  issuer: C=US; O=Let's Encrypt; CN=R10
*  SSL certificate verify ok.
} [5 bytes data]
* using HTTP/2
* h2h3 [:method: GET]
* h2h3 [:path: /api/health]
* h2h3 [:scheme: https]
* h2h3 [:authority: dev.capitabyte.com]
* h2h3 [user-agent: curl/7.88.1]
* h2h3 [accept: */*]
* Using Stream ID: 1 (easy handle 0x55bcbe49f7a0)
} [5 bytes data]
> GET /api/health HTTP/2
> Host: dev.capitabyte.com
> user-agent: curl/7.88.1
> accept: */*
>
{ [5 bytes data]
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
{ [57 bytes data]
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
{ [57 bytes data]
* old SSL session ID is stale, removing
{ [5 bytes data]
< HTTP/2 200
< date: Wed, 20 Aug 2025 07:14:57 GMT
< content-type: text/html; charset=utf-8
< content-length: 2
< x-powered-by: Express
< etag: W/"2-nOO9QiTIwXgNtWtBJezz8kv3SLc"
< strict-transport-security: max-age=31536000; includeSubDomains
<
{ [2 bytes data]
* Connection #0 to host dev.capitabyte.com left intact
OK
```

## Acceptance

- Both internal and external requests return HTTP 200.
