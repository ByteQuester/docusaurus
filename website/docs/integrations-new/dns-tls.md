---
id: dns-and-tls
slug: /integrations/dns-and-tls
sidebar_position: 8
---

# DNS and TLS

:::info This document outlines how we manage our application's domain, DNS records, and TLS certificates. :::

## Overview

Our strategy for managing DNS and TLS involves four main components working together in our EKS cluster:

1.  **AWS Route 53:** Our authoritative DNS provider.
2.  **ExternalDNS:** A Kubernetes controller that automatically manages Route 53 records based on our Ingress resources.
3.  **cert-manager:** A controller that automates the management and issuance of TLS certificates from Let's Encrypt.
4.  **Ingress NGINX:** Our Ingress controller that routes external traffic and terminates TLS.

## Automation

### DNS Automation with ExternalDNS

Instead of manually creating `A` records in Route 53, we automate the process using ExternalDNS.

- ExternalDNS watches the Kubernetes API for Ingress resources.
- When an Ingress with a `host` is created or changed, ExternalDNS automatically creates or updates the corresponding `A` record in Route 53 to point to the load balancer for our Ingress NGINX controller.

:::tip IRSA for ExternalDNS For this to work, ExternalDNS needs permission to modify Route 53 records. We provide this securely using IAM Roles for Service Accounts (IRSA). For a detailed explanation, see **[IRSA for ExternalDNS](./irsa-externaldns.md)**. :::

### TLS Automation with cert-manager

We use `cert-manager` to automatically issue and renew TLS certificates from Let's Encrypt, so we never have to handle private keys manually.

#### ClusterIssuers

We have two `ClusterIssuer` resources defined in `infra/manifests/cert-manager/clusterissuers.yaml`: one for staging and one for production.

- `letsencrypt-staging`: Used for development and testing to avoid hitting Let's Encrypt rate limits.
- `letsencrypt-prod`: Used for production workloads to issue trusted certificates.

```yaml title="infra/manifests/cert-manager/clusterissuers.yaml"
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod-private-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

#### Requesting a Certificate

To get a certificate, you simply add an annotation to your Ingress resource specifying which issuer to use, and `cert-manager` handles the rest.

## Example: Tying It All Together

Here is an example of a Helm `values.yaml` snippet for an application that configures an Ingress to get both an automatic DNS record and a TLS certificate.

```yaml title="charts/frontend-sample/values.yaml"
ingress:
  enabled: true
  className: 'nginx'
  annotations:
    cert-manager.io/cluster-issuer: 'letsencrypt-prod'
  hosts:
    - host: dev.capitabyte.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: dev-capitabyte-com-tls
      hosts:
        - dev.capitabyte.com
```

When this Ingress is deployed:

1.  **ExternalDNS** sees `dev.capitabyte.com` and creates a Route 53 record pointing to the ingress controller.
2.  **cert-manager** sees the `cert-manager.io/cluster-issuer` annotation and the `tls` block. It solves the ACME challenge and creates a certificate, storing it in the `dev-capitabyte-com-tls` Secret.
3.  **Ingress NGINX** loads the secret and terminates TLS for traffic to `dev.capitabyte.com`.

## DNS & TLS Setup Checklist

:::note Troubleshooting If you encounter issues with the cert-manager installation (e.g., pods not starting, certificates not being issued), please refer to the [TLS Troubleshooting](../runbooks/tls-troubleshooting.md) guide. The most common issue is an inconsistent kube context between `helm` and `kubectl`. :::

<details>
  <summary>1. Prerequisites</summary>

- [ ] `ingress-nginx` is installed in your cluster.
- [ ] `cert-manager` is installed and running in your cluster.
- [ ] You have a domain name registered with a DNS provider.
- [ ] You have a Route 53 Hosted Zone for your domain.

</details>

<details>
  <summary>2. DNS Configuration</summary>

Your domain needs to point to the external IP address of the `ingress-nginx` load balancer.

- **Find the Load Balancer IP:**

  ```sh
  kubectl get svc -n ingress ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
  ```

- **Create DNS Record:**
  - **Manual:** Create an `A` or `ALIAS` record in your Route 53 hosted zone pointing `dev.capitabyte.com` to the address from the previous command.
  - **ExternalDNS:** This is handled automatically if ExternalDNS is configured.

</details>

<details>
  <summary>3. Issue a Staging Certificate (Recommended)</summary>

First, test against the Let's Encrypt staging environment to avoid rate limits.

1.  **Annotate Ingress:** In `charts/frontend-sample/values.yaml`, configure the Ingress with the staging issuer:

    ```yaml
    ingress:
      annotations:
        cert-manager.io/cluster-issuer: letsencrypt-staging
      tls:
        - hosts:
            - dev.capitabyte.com
          secretName: dev-capitabyte-com-tls
    ```

2.  **Deploy:** Deploy your application using `helm`.

    ```sh
    helm upgrade --install frontend-sample charts/frontend-sample --namespace frontend-dev
    ```

3.  **Verify Certificate:** Check the status of the certificate. It should become `Ready` within a few minutes.
    ```sh
    kubectl -n frontend-dev get certificate dev-capitabyte-com-tls -o yaml
    ```

</details>

<details>
  <summary>4. Promote to Production Certificate</summary>

Once the staging certificate is successful, switch to the production issuer.

1.  **Update Annotation:** Change the issuer in `charts/frontend-sample/values.yaml`:

    ```yaml
    ingress:
      annotations:
        cert-manager.io/cluster-issuer: letsencrypt-prod
    ```

2.  **Redeploy:** Deploy the change.

    ```sh
    helm upgrade --install frontend-sample charts/frontend-sample --namespace frontend-dev
    ```

3.  **Verify Certificate:** The certificate will be re-issued. Check its status again.
    ```sh
    kubectl -n frontend-dev get certificate dev-capitabyte-com-tls
    ```

</details>

<details>
  <summary>5. Final Verification</summary>

- **cURL Test:** A successful `curl` command confirms a valid TLS certificate.

  ```sh
  curl -I https://dev.capitabyte.com
  ```

  _Expected Output:_

  ```
  HTTP/2 200
  date: Sat, 16 Aug 2025 11:42:02 GMT
  content-type: text/html
  ...
  ```

- **OpenSSL Test (Detailed):** For more details on the certificate chain.
  ```sh
  openssl s_client -connect dev.capitabyte.com:443 -servername dev.capitabyte.com
  ```

</details>
