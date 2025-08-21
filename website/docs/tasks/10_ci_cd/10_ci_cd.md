---
id: ci-cd-and-supply-chain
slug: /tasks/10-ci-cd/ci-cd-and-supply-chain
sidebar_position: 1
---

# CI/CD and Supply Chain

### CI-001 — Bootstrap CI (lint, type-check, build)

- **Dependencies:** REPO-003, REPO-004
- **Deliverables:** `.github/workflows/ci.yml` with install cache, lint, type-check, build for `apps/frontend`.
- **Acceptance:** CI green on PR.

### CI-002 — Build and push images to GHCR

- **Dependencies:** CI-001
- **Deliverables:** workflow to build and push `apps/frontend` image to GHCR on `main` with tags (`sha`, `latest`, `dev-<shortsha>`).
- **Acceptance:** image visible in Packages; pull works locally.

### CI-003 — Helm lint and template checks

- **Dependencies:** HELM-001
- **Deliverables:** CI jobs for `helm lint` and `helm template` with sample values.
- **Acceptance:** CI fails on invalid charts; passes when fixed.

### CI-004 — Security scanning baseline

- **Dependencies:** CI-001
- **Deliverables:** Gitleaks (secrets), Trivy (image and fs), Checkov (IaC) jobs; baseline config in `policy/`.
- **Acceptance:** seeded test issues fail; cleaned build passes.

### CI-005 — Caching and performance

- **Dependencies:** CI-001
- **Deliverables:** optimized node/pnpm and Docker layer caching.
- **Acceptance:** rerun time decreases on no-op commits.

### CI-006 — Matrix builds for apps

- **Dependencies:** CI-001
- **Deliverables:** matrix strategy to detect and build only changed apps.
- **Acceptance:** PR touching only one app builds that app.

### CI-007 — Release versioning and changelogs

- **Dependencies:** CI-002, HELM-003
- **Deliverables:** tag-driven releases; changelog generation; version bump for charts and images.
- **Acceptance:** tag creates release artifacts with correct versions.

### CI-008 — SBOM generation and upload

- **Dependencies:** CI-002
- **Deliverables:** generate SBOM (Syft) for images; upload as release asset.
- **Acceptance:** SBOM attached to release; retrievable.

### CI-009 — Image signing

- **Dependencies:** CI-002
- **Deliverables:** cosign sign images; store signatures; document verification.
- **Acceptance:** `cosign verify` succeeds for published images.

### CI-010 — Branch protections

- **Dependencies:** CI-001
- **Deliverables:** document required checks; enable in repo settings (manual step noted).
- **Acceptance:** protected branches enforce status checks.

### CI-011 — Align Node version file

- **Dependencies:** CI-001
- **Deliverables:** ensure workflow uses the repo's Node version file. Either add `.nvmrc` matching `.node-version` or update workflow to reference `.node-version`.
- **Acceptance:** CI sets the intended Node version; build/test succeed.

### CI-012 — Add missing policy configs

- **Dependencies:** CI-004
- **Deliverables:** add `policy/.gitleaks.toml`, `policy/.trivyignore`, and `policy/.checkov.yaml` with sensible defaults and documented thresholds.
- **Acceptance:** CI scanners run without missing-file errors; fail only on real issues.
