---
id: zero-to-hero-guided-path
slug: /guides/zero-to-hero-guided-path
sidebar_position: 1
---

# Zero-to-Hero Guided Path

:::info This roadmap sequences the tech stack (Ingress NGINX, cert-manager, Argo CD, Kyverno, Prometheus/Grafana, NetworkPolicies, RBAC) into practical phases. It reuses your existing tasks/docs and adds validation gates so you don’t advance while a component is not working as expected. :::

:::danger Important Always use `--kubeconfig=$HOME/.kube/config` in `kubectl` commands. :::

## Phase 0 — Minimal Reset & Sample Redeploy (Day-1 unblock)

- **Read:** `docs/docs/fresh-start-redeploy.md`
- **Goal:** Backend + Frontend green internally (ClusterIP) and externally (Ingress). No custom NetworkPolicies yet; chart-managed ServiceAccounts.
- **Gate:** Both internal curl and external `/api/health` return 200.

## Phase 1 — Core Platform

- [Ingress NGINX (AWSZ-007)](/tasks/00-setup/aws-zero-to-hero#awsz-007): `infra/manifests/ingress-nginx/values.yaml`, verify controller Ready.
- [cert-manager (AWSZ-008)](/tasks/00-setup/aws-zero-to-hero#awsz-008): install CRDs + controller; staging first; clusterissuers present.
- [ExternalDNS (AWSZ-009)](/tasks/00-setup/aws-zero-to-hero#awsz-009): can defer; DNS records automated later.
- [DNS configuration (AWSZ-010)](/tasks/00-setup/aws-zero-to-hero#awsz-010): domain points to ingress LB.
- **Gate:** `nslookup dev.<domain>` resolves; HTTPS padlock later (AWSZ-018/019).

## Phase 2 — Observability core

- [Prometheus stack (AWSZ-024)](/tasks/00-setup/aws-zero-to-hero#awsz-024): `docs/docs/prometheus-stack-install.md`.
- [Uptime rules (AWSZ-025)](/tasks/00-setup/aws-zero-to-hero#awsz-025): `infra/manifests/prometheus/rules/uptime.yaml`.
- [Grafana dashboards (CAP-001)](/tasks/capacity/README.md): `docs/docs/grafana-dashboards.md`.
- **Gate:** Prometheus targets Up, stock dashboards show data.

## Phase 3 — Sample apps deploy

- [Backend-sample (AWSZ-016)](/tasks/00-setup/aws-zero-to-hero#awsz-016) → `backend-dev`.
- [Frontend-sample (AWSZ-015)](/tasks/00-setup/aws-zero-to-hero#awsz-015) → `frontend-dev`.
- [Wire (AWSZ-017)](/tasks/00-setup/aws-zero-to-hero#awsz-017): `VITE_API_BASE_URL` to backend.
- **Gate:** internal and external health OK; screenshots in `docs/docs/portfolio.md`.

## Phase 4 — Security baseline (Kyverno + RBAC)

- [Kyverno baseline (AWSZ-027)](/tasks/00-setup/aws-zero-to-hero#awsz-027): `infra/manifests/kyverno-policies/policies.yaml` with staged `validationFailureAction` (audit → enforce after validation).
- **Gate:** No RBAC Forbidden in app logs; Kyverno policies aligned with repo intent.

## Phase 5 — NetworkPolicies baseline (AWSZ-028)

- [Apply minimal set only (AWSZ-028)](/tasks/00-setup/aws-zero-to-hero#awsz-028):
  - frontend-dev: `00-deny-all.yaml`, `allow-dns.yaml`
  - backend-dev: `00-deny-all.yaml`, `allow-dns.yaml`, `10-allow-from-frontend.yaml`, `15-allow-from-ingress.yaml`
- Do not add policies in `kube-system`.
- **Gate:** Internal and external health remain 200 under policies.

## Phase 6 — GitOps (Argo CD)

- [Install Argo CD (AWSZ-020)](/tasks/00-setup/aws-zero-to-hero#awsz-020) and [fix destination (AWSZ-021)](/tasks/00-setup/aws-zero-to-hero#awsz-021).
- [Manage apps via GitOps (AWSZ-022/023)](/tasks/00-setup/aws-zero-to-hero#awsz-022).
- **Gate:** Apps Healthy/Synced; drift corrected by Argo.

## Phase 7 — TLS and polish

- [TLS staging (AWSZ-018)](/tasks/00-setup/aws-zero-to-hero#awsz-018) → [production (AWSZ-019)](/tasks/00-setup/aws-zero-to-hero#awsz-019).
- [Finalize portfolio links, screenshots (AWSZ-031)](/tasks/00-setup/aws-zero-to-hero#awsz-031): `docs/docs/portfolio.md`, `docs/docs/integration-links.md`.
- **Gate:** HTTPS padlock; links rendered.

## Phase 8 — Capacity and performance

- [CAP-002 alerts; CAP-003 audit (done)](/tasks/capacity/README.md) → `docs/docs/capacity-audit.md`.

## Guardrails and conventions

- Single enforcement engine for NetworkPolicy (Calico). No kube-system netpols.
- No ad-hoc manifests at repo root; policies live under `infra/manifests/...`; debug YAMLs under `ops/`.
- Always cite tasks/docs from PRs; small, focused branches.

## If trouble strikes
