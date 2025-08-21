---
id: zero-to-hero
slug: /guides/zero-to-hero
sidebar_position: 20
---

# Zero to Hero: First Monorepo Deployment

:::info This guide is for developers new to this monorepo. The goal is to get from a fresh clone to a live frontend and backend-sample running on a cloud cluster with CI proofs.

For a more detailed, atomic task list, see the [AWS Zero-to-Hero Tasks](/tasks/00-setup/aws-zero-to-hero). :::

| Prerequisite | Details - |
| --- | --- |
| Cloud Account | A cloud account (or a single Linux VM) and a domain (optional for first HTTP test) - |
| Local Tools | `kubectl`, `Helm`, `pnpm`, `Docker` ... [truncated] |
| Docker Login | Logged in to a container registry (e.g., `docker login ghcr.io`) ... [truncated] |

## 1) Clone and Install

- **Action**: clone repo, install deps
  - `pnpm install`
- **Acceptance**: `pnpm -w lint` and `pnpm -w build` succeed
- **Related Task**: [AWSZ-001 — Prereqs and local toolchain](/tasks/00-setup/aws-zero-to-hero#awsz-001--prereqs-and-local-toolchain)

## 2) Verify CI Locally (Optional)

- **Action**: run frontend tests
  - `pnpm --filter vite_react_shadcn_ts test`
- **Acceptance**: tests pass

## 3) Build Images

- **Action**: build frontend and backend-sample images locally
  - Frontend: `docker build -f apps/frontend/Dockerfile -t ghcr.io/<owner>/<repo>:dev .`
  - Backend-sample: `docker build -f apps/backend-sample/Dockerfile -t ghcr.io/<owner>/<repo>/backend-sample:dev apps/backend-sample`
- **Acceptance**: images built
- **Related Task**: [AWSZ-013 — Frontend image push](/tasks/00-setup/aws-zero-to-hero#awsz-013--frontend-image-push)

## 4) Pick a Cluster Path

- **Option A (fastest)**: single VM with k3s
  - Install k3s: `curl -sfL https://get.k3s.io | sh -`
  - `kubectl get nodes` shows Ready
- **Option B (managed)**: EKS/GKE (follow provider docs or Terraform modules under `infra/modules/cluster`)
- **Acceptance**: `kubectl get nodes` Ready
- **Related Task**: [AWSZ-005 — Provision EKS cluster (Terraform)](/tasks/00-setup/aws-zero-to-hero#awsz-005--provision-eks-cluster-terraform)

## 5) Install Ingress Controller

- **Action**: install ingress-nginx
  - `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`
  - `helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx -n ingress --create-namespace -f infra/manifests/ingress-nginx/values.yaml`
- **Acceptance**: controller pods Ready; LB address available (managed) or NodePort (k3s)
- **Related Task**: [AWSZ-007 — Ingress Controller (ingress-nginx)](/tasks/00-setup/aws-zero-to-hero#awsz-007--ingress-controller-ingress-nginx)

## 6) Install Cert-Manager (Optional)

- **Action**: install cert-manager (staging issuer)
  - `helm repo add jetstack https://charts.jetstack.io`
  - `helm upgrade --install cert-manager jetstack/cert-manager -n cert-manager --create-namespace --set crds.install=true`
  - `kubectl apply -f infra/manifests/cert-manager/clusterissuers.yaml`
- **Acceptance**: issuers Ready
- **Related Task**: [AWSZ-008 — cert-manager (CRDs + controller)](/tasks/00-setup/aws-zero-to-hero#awsz-008--cert-manager-crds--controller)

## 7) Deploy Frontend via Helm

- **Action**: configure values and deploy
  - Set image values: `--set image.repository=ghcr.io/<owner>/<repo> --set image.tag=dev`
  - Deploy: `helm upgrade --install frontend charts/frontend -n frontend-dev --create-namespace -f charts/frontend/values-dev.yaml --set image.repository=ghcr.io/<owner>/<repo> --set image.tag=dev`
- **Acceptance**: `kubectl get ingress -n frontend-dev` shows address; `curl http://<address>/` returns 200
- **Related Task**: [AWSZ-015 — Frontend deploy (Helm, dev)](/tasks/00-setup/aws-zero-to-hero#awsz-015--frontend-deploy-helm-dev)

## 8) Deploy Backend-Sample via Helm

- **Action**: deploy backend-sample under same host at `/api`
  - `helm upgrade --install backend-sample charts/backend-sample -n backend-dev --create-namespace --set image.repository=ghcr.io/<owner>/<repo>/backend-sample --set image.tag=dev --set ingress.hosts[0].host=<same host>`
- **Acceptance**: `curl http://<address>/api/health` returns 200
- **Related Task**: [AWSZ-016 — Backend-sample deploy (Helm, dev)](/tasks/00-setup/aws-zero-to-hero#awsz-016--backend-sample-deploy-helm-dev)

## 9) Wire Frontend → Backend

- **Action**: set API base URL and redeploy frontend
  - Choose: build-time env (rebuild image) or ConfigMap/env and template in chart
  - Example helm set: `--set configMap.data.VITE_API_BASE_URL=http://<address>/api`
- **Acceptance**: frontend displays data fetched from `/api`
- **Related Task**: [AWSZ-017 — Wire frontend → backend](/tasks/00-setup/aws-zero-to-hero#awsz-017--wire-frontend--backend)

## 10) Optional: DNS + TLS

- **Action**: point `dev.<domain>` to LB; apply production issuer
  - Update frontend values: `--set ingress.hosts[0].host=dev.<domain>`
- **Acceptance**: HTTPS padlock visible
- **Related Tasks**:
  - [AWSZ-010 — Domain and DNS](/tasks/00-setup/aws-zero-to-hero#awsz-010--domain-and-dns-manual-or-via-externaldns)
  - [AWSZ-018 — TLS staging](/tasks/00-setup/aws-zero-to-hero#awsz-018--tls-staging)
  - [AWSZ-019 — TLS production](/tasks/00-setup/aws-zero-to-hero#awsz-019--tls-production)

## 11) Optional: GitOps

- **Action**: fix `infra/gitops/apps/frontend.yaml` destination URL; install Argo CD; apply Application
- **Acceptance**: Argo shows app Healthy/Synced
- **Related Tasks**:
  - [AWSZ-020 — Argo CD install](/tasks/00-setup/aws-zero-to-hero#awsz-020--argo-cd-install)
  - [AWSZ-021 — Fix Argo Application destination URL](/tasks/00-setup/aws-zero-to-hero#awsz-021--fix-argo-application-destination-url)
  - [AWSZ-022 — GitOps manage frontend (optional)](/tasks/00-setup/aws-zero-to-hero#awsz-022--gitops-manage-frontend-optional)

## 12) CI/CD Proofs

- **Action**: push to `main` and confirm GHCR image push; PRs trigger lint/test/helm
- **Acceptance**: GHCR images appear; CI green; scanners run without missing configs

## 13) Document and Capture Proofs

- **Action**: update `../integrations/integration-links.md` with live links and screenshots (Argo/Grafana if available)
- **Acceptance**: links/screenshots present for portfolio
- **Related Task**: [AWSZ-031 — Portfolio links](/tasks/00-setup/aws-zero-to-hero#awsz-031--portfolio-links)

---

### Troubleshooting

- **No ingress address?** Check controller namespace pods and service type
- **404 on `/api/health`?** Confirm backend ingress host/path matches frontend host
- **TLS fails?** Use staging issuer first; check cert-manager events
