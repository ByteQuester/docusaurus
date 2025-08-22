---
id: blueprints-and-templates
slug: /tasks/15-blueprints-and-tooling/blueprints-and-templates
sidebar_position: 1
---

# Blueprints and Templates (Reproducible Onboarding)

:::info

Atomic tasks to create reusable templates so adding N frontends/backends is fast, consistent, and infra-safe.

:::

### BP-001 — Helm chart template for services

- **Dependencies:** HELM-001
- **Deliverables:** `charts/service-template/` with Deployment, Service, Ingress (path and host), HPA, ConfigMap, ServiceAccount; README with usage; placeholders for name/image/paths.
- **Acceptance:** `helm lint` passes; `helm template` renders with sample values.

### BP-002 — Environment values pattern

- **Dependencies:** BP-001
- **Deliverables:** `values-dev.yaml` and `values-prod.yaml` templates including image tag, host, resources, HPA, annotations; documented keys.
- **Acceptance:** renders both envs; no secrets.

### BP-003 — Reusable CI workflow for build/push

- **Dependencies:** CI-002
- **Deliverables:** `.github/workflows/reusable-build-push.yml` that accepts inputs (app_path, dockerfile_path, image_name); example caller workflow in each app.
- **Acceptance:** calling workflow builds and pushes image for the specified app.

### BP-004 — Reusable CI workflow for chart lint/template

- **Dependencies:** CI-003
- **Deliverables:** `.github/workflows/reusable-helm-validate.yml` that accepts chart_path and values list.
- **Acceptance:** caller job validates charts by path.

### BP-005 — Terraform IRSA generic module

- **Dependencies:** INF-001
- **Deliverables:** `infra/modules/iam/irsa-generic/` module to create IAM policy + role + OIDC trust for a given ServiceAccount (namespace/name) and policy JSON.
- **Acceptance:** `terraform validate` passes; example usage doc included.

### BP-006 — ExternalDNS IRSA via module

- **Dependencies:** BP-005, AWSZ-009-TF
- **Deliverables:** Instantiate `irsa-generic` for ExternalDNS with AWS-managed policy JSON; document outputs; update Helm values to use the role ARN annotation.
- **Acceptance:** external-dns pod assumes role; DNS updates succeed.

### BP-007 — cert-manager DNS01 IRSA (optional)

- **Dependencies:** BP-005
- **Deliverables:** Module instantiation for cert-manager Route 53 DNS01 if used; values and annotations documented.
- **Acceptance:** ACME DNS01 challenges succeed.

### BP-008 — GitOps Application template

- **Dependencies:** GITOPS-003
- **Deliverables:** `infra/gitops/apps/_template-app.yaml` with placeholders (app name, chart path, values files, destination namespace).
- **Acceptance:** copying and replacing placeholders yields a valid Application.

### BP-009 — Service onboarding checklist and scaffold script

- **Dependencies:** REPO-003, BP-001..BP-004
- **Deliverables:** `docs/docs/service-onboarding.md` with a numbered checklist; `scripts/scaffold/new-service.sh` to copy templates and replace placeholders.
- **Acceptance:** running script creates `apps/<name>` and `charts/<name>` from templates; checklists render in docs.

### BP-010 — Path-based ingress strategy for unified site

- **Dependencies:** BP-001, HELM-001
- **Deliverables:** Document and template how multiple backends are mounted under `/api/<service>` while frontend serves `/` on the same host; include CORS guidance.
- **Acceptance:** template values show example paths; ingress renders correctly.

---

### BP-011 — Reusable k6 test template

- **Dependencies:** OPS-001
- **Deliverables:** `ops/k6/template.js` with env-driven base URL and threshold placeholders; README for duplication per service.
- **Acceptance:** running template against any service works with env vars.

### BP-012 — Portfolio doc template entry

- **Dependencies:** DOC-007
- **Deliverables:** snippet template to add a service to the portfolio page with links (ingress path, repo, chart, dashboard).
- **Acceptance:** pasting the snippet yields consistent docs entries.
