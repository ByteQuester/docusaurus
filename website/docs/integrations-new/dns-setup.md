---
id: dns-setup
slug: /integrations/dns-setup
sidebar_position: 6
---

# DNS and Domain Configuration

:::info This guide explains how to point your domain to the Ingress controller's load balancer, enabling access to your applications. :::

## 1. Find the Ingress Load Balancer IP

After installing the Ingress controller (e.g., NGINX), it will provision a cloud load balancer. You need to get the external IP address of this load balancer.

```bash title="Get Ingress Controller Service"
kubectl get services --namespace ingress-nginx
```

:::tip IP Assignment It might take a few minutes for the `EXTERNAL-IP` to be assigned. :::

Look for the `ingress-nginx-controller` service. The external IP will be listed in the `EXTERNAL-IP` column.

```
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
ingress-nginx-controller   LoadBalancer   10.10.10.10     35.xxx.xxx.xxx   80:31234/TCP,443:54321/TCP   5m
```

In this example, the external IP is `35.xxx.xxx.xxx`.

## 2. Configure Your DNS Provider

Now, you need to go to your DNS provider (e.g., Google Domains, Cloudflare, AWS Route 53, etc.) and create a DNS `A` record.

| Field | Value |
| --- | --- |
| **Record Type** | `A` |
| **Host/Name** | `@` (for the root domain) or a subdomain like `www` or `app`. |
| **Value/Points to** | The external IP address of your load balancer (e.g., `35.xxx.xxx.xxx`). |
| **TTL** | Leave the default value. |

:::note Wildcard Domains If you want to use a wildcard domain (e.g., `*.example.com`), you can create a wildcard `A` record. :::

## 3. Verification

After the DNS record has propagated (which can take some time), you can verify that your domain resolves to the correct IP address using a tool like `dig` or `nslookup`.

```bash title="Verify DNS Propagation"
dig my-app.example.com
```

:::success The output should show the `A` record pointing to your load balancer's IP address. Once this is configured, any Ingress resources you create with a matching hostname will be accessible at that domain. :::
