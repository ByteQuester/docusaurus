---
id: cert-manager-recovery
slug: /runbooks/cert-manager-recovery
sidebar_position: 5
---

# Cert-Manager Recovery (Atomic Tasks)

:::info This runbook provides a set of atomic tasks to resolve inconsistent Helm/CRD state for cert-manager and complete the TLS setup. :::

:::danger Important Only proceed if you have cluster-admin access. If this is a shared cluster with existing cert-manager consumers, coordinate with your team first. :::

<details>
  <summary>CERT-000 — Align kube context (avoid sudo mismatch)</summary>

- **Dependencies:** AWSZ-006
- **Deliverables:** Ensure helm and kubectl point to the same cluster/context and kubeconfig.
- **Steps:**
  - Print context as the current user:
    ```bash
    kubectl config current-context && kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}' | cat
    ```
  - Print context as root:
    ```bash
    sudo env -i KUBECONFIG=$KUBECONFIG kubectl config current-context || true
    ```
  - Decide on one kubeconfig path (e.g., `$HOME/.kube/config`) and export `KUBECONFIG` explicitly in your shell before running ANY commands.
  - For every helm command, pass `--kubeconfig $KUBECONFIG` (same file).
  - Do NOT use `sudo` with kubectl unless you also pass the same `--kubeconfig`.
- **Acceptance:** `helm ls -A --kubeconfig $KUBECONFIG` and `kubectl get ns --kubeconfig $KUBECONFIG` show consistent clusters.

</details>

<details>
  <summary>CERT-001 — Snapshot current state</summary>

- **Dependencies:** AWSZ-006
- **Deliverables:** Collect state for later reference and rollback if needed.
- **Steps:**
  ```bash
  helm list -A | grep cert-manager | cat
  kubectl get ns cert-manager -o yaml | cat
  kubectl get deploy,sts,ds,po,svc,sa,role,rolebinding -n cert-manager | cat
  kubectl get crd | grep cert-manager.io | cat
  kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations | grep cert-manager | cat
  kubectl get events -A | grep -i cert-manager | cat
  ```
- **Acceptance:** Evidence captured in a gist or `./tls-troubleshooting.md`.

</details>

<details>
  <summary>CERT-002 — Uninstall existing release (if present)</summary>

- **Dependencies:** CERT-001
- **Steps:**
  - If `helm ls` shows a release, run:
    ```bash
    helm uninstall cert-manager -n cert-manager || true
    ```
  - Wait for namespace objects to terminate.
- **Acceptance:** `helm status cert-manager -n cert-manager` shows release not found.

</details>

<details>
  <summary>CERT-003 — Purge Helm release secrets (Helm v3)</summary>

- **Dependencies:** CERT-002
- **Steps:**
  ```bash
  kubectl -n cert-manager delete secret -l owner=helm,name=cert-manager || true
  ```
- **Acceptance:** No secrets named `sh.helm.release.v1.cert-manager.*` remain in cert-manager ns.

</details>

<details>
  <summary>CERT-004 — Remove stale CRDs (only if safe)</summary>

- **Dependencies:** CERT-001
- **Preconditions:** Confirm no in-cluster resources rely on cert-manager (Certificates, Issuers). If this is a fresh setup, proceed.
- **Steps:**
  - For each listed CRD (e.g., `certificates.cert-manager.io`, `issuers.cert-manager.io`, etc.):
    ```bash
    kubectl delete crd <crd-name> || true
    ```
- **Acceptance:** `kubectl get crd | grep cert-manager.io` returns nothing.

</details>

<details>
  <summary>CERT-005 — Remove stale webhook configs (if present)</summary>

- **Dependencies:** CERT-001
- **Steps:**
  ```bash
  kubectl delete validatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook || true
  kubectl delete mutatingwebhookconfiguration.admissionregistration.k8s.io cert-manager-webhook || true
  ```
- **Acceptance:** No webhook configurations referencing cert-manager remain.

</details>

<details>
  <summary>CERT-006 — Add repo and install with CRDs (pinned version)</summary>

- **Dependencies:** CERT-003
- **Steps:**
  - ```bash
    helm repo add jetstack https://charts.jetstack.io && helm repo update
    ```
  - Choose a version compatible with your K8s (example: `--version v1.14.5` for broad compatibility, or stick to `v1.18.x` on newer clusters).
  - Install with CRDs enabled:
    ```bash
    helm upgrade --install cert-manager jetstack/cert-manager \
      -n cert-manager --create-namespace \
      --kubeconfig $KUBECONFIG \
      --version v1.14.5 \
      --set crds.enabled=true
    ```
- **Acceptance:**
  - `kubectl -n cert-manager get pods` shows all pods Ready (controller, cainjector, webhook)
  - `kubectl get crd | grep cert-manager.io` lists expected CRDs

</details>

<details>
  <summary>CERT-007 — Basic health checks</summary>

- **Dependencies:** CERT-006
- **Steps:**
  ```bash
  kubectl -n cert-manager logs deploy/cert-manager --tail=200 | cat
  kubectl -n cert-manager logs deploy/cert-manager-webhook --tail=200 | cat
  kubectl get apiservices | grep cert-manager || true
  ```
- **Acceptance:** No crash loops; webhook listens; controller reconciling.

</details>

<details>
  <summary>CERT-008 — Install staging ClusterIssuer</summary>

- **Dependencies:** CERT-006
- **Steps:**
  - Review `infra/manifests/cert-manager/clusterissuers.yaml`. Ensure email and HTTP-01 solver (ingressClass: nginx) are set for staging and prod.
  - ```bash
    kubectl apply -f infra/manifests/cert-manager/clusterissuers.yaml
    kubectl get clusterissuer letsencrypt-staging -o yaml | cat
    ```
- **Acceptance:** ClusterIssuer `letsencrypt-staging` Ready=True.

</details>

<details>
  <summary>CERT-009 — Verify HTTP-01 reachability</summary>

- **Dependencies:** AWSZ-007, AWSZ-015 or AWSZ-015F
- **Steps:**
  - Ensure your frontend Ingress serves HTTP on port 80 without forced redirect to HTTPS during challenge.
  - Confirm Ingress has correct class (nginx) and host set to your dev domain.
  - ```bash
    curl -I http://dev.<your-domain>/.well-known/acme-challenge/test
    ```
- **Acceptance:** During issuance, cert-manager creates temporary challenge Ingress; events show solver success.

</details>

<details>
  <summary>CERT-010 — Annotate Ingress for certificate</summary>

- **Dependencies:** CERT-008, CERT-009
- **Steps:**
  - Ensure frontend Ingress has:
    - `metadata.annotations: cert-manager.io/cluster-issuer: letsencrypt-staging` (then prod)
    - `spec.tls: hosts: [dev.<your-domain>]` and `secretName: dev-tls`
  - Apply Helm values accordingly in `charts/frontend(-sample)/values-dev.yaml`.
- **Acceptance:** `kubectl -n frontend-dev get certificate` shows Ready=True; secret created.

</details>

<details>
  <summary>CERT-011 — Promote to production issuer</summary>

- **Dependencies:** CERT-010
- **Steps:**
  - Switch annotation to `cert-manager.io/cluster-issuer: letsencrypt-prod`
  - Apply; watch events and certificate Ready=True.
- **Acceptance:** HTTPS padlock valid at `https://dev.<your-domain>`

</details>

<details>
  <summary>CERT-012 — Document troubleshooting and outcome</summary>

- **Dependencies:** CERT-011
- **Deliverables:** Update `./tls-troubleshooting.md` with commands, outputs, and what fixed the issue.
- **Acceptance:** Doc contains before/after and can be reused by future agents.

</details>
