# DNS-FIX-001 — Cleanup stray files and temporary policies

- Status: In Progress
- Dependencies: DNS-TRIAGE-003

## Scope

Normalize ad-hoc YAMLs created at repo root and move legitimate artifacts into the right directories; delete the rest.

## Actions

- Review the following files at repo root and take action:
  - `debugging_report.md` → move to `tasks/incidents/debugging_report_dns.md` (rename and keep as incident record).
  - `ingress-nginx-admission-webhook.yaml` → do not commit cluster-derived objects; remove from repo.
  - `frontend-service.yaml`, `frontend-service-modified.yaml` → remove; service should be managed by Helm/Argo.
  - `debug-pod.yaml`, `debug-pod-kube-system.yaml`, `debug-pod-frontend.yaml` → move to `ops/debug/` as examples.
  - `allow-ingress-webhook.yaml`, `allow-from-ingress.yaml`, `allow-from-ingress-modified.yaml`, `allow-from-default-for-debug.yaml`, `allow-egress-to-ingress-webhook.yaml` → remove or migrate to `infra/manifests/network-policies/` only if truly required and aligned with policy model; prefer deletion to avoid drift.

## Acceptance

- Root directory free of ad-hoc manifests; any retained examples live under `ops/`.
- NetworkPolicy manifests reside only under `infra/manifests/network-policies/` following naming conventions.
