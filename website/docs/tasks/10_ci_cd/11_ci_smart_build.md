---
id: ci-smart-build
slug: /tasks/10-ci-cd/ci-smart-build
sidebar_position: 2
---

# Smart CI for Monorepo (Atomic Tasks)

:::info **Goal:** Make CI detect changed apps and charts reliably, run the right jobs, and avoid skip-chains. :::

### CI-013 — Add path filters for apps and charts

- **Dependencies:** CI-001
- **Deliverables:** In `.github/workflows/ci.yml`, use `dorny/paths-filter@v3` to output two JSON arrays: `apps_files` (apps/**) and `charts_files` (charts/**). Expose both as job outputs.
- **Acceptance:** Debug step prints non-empty arrays when touching files under `apps/` or `charts/`.

### CI-014 — Decouple Helm chart tests from app builds

- **Dependencies:** CI-013
- **Deliverables:**
  - Make `helm-test` depend on the checkout step (or `determine-changes`), not on `build`.
  - Add `if:` to `helm-test` to run when `charts_files` is not empty OR on pushes to main.
- **Acceptance:** Editing only `charts/frontend/**` triggers `helm-test` even if no apps changed.

### CI-015 — Matrix build for changed apps

- **Dependencies:** CI-013
- **Deliverables:** Ensure `build` job matrix uses the `apps` list computed from `apps_files`. Keep existing pnpm filter usage.
- **Acceptance:** Editing `apps/frontend-sample/**` runs a build only for `frontend-sample`.

### CI-016 — Fallback behavior (optional)

- **Dependencies:** CI-015
- **Deliverables:** On PRs with no detected app changes, optionally build a small default target (e.g., `frontend-sample`) to keep coverage.
- **Acceptance:** PR with unrelated changes still validates at least one app if fallback enabled.

### CI-017 — Security scanners not blocked by skip-chain

- **Dependencies:** CI-013
- **Deliverables:** Consider splitting scanners into their own job depending only on checkout; or guard `security_and_publish` with conditions so it runs on main and tags regardless of `build`.
- **Acceptance:** Scanners run on main push even when only infra or docs changed.

### CI-018 — Artifacts and logs for debugging

- **Dependencies:** none
- **Deliverables:** Upload computed `apps.json` and `charts.json` as workflow artifacts; echo diagnostic info.
- **Acceptance:** Artifacts visible in Actions UI for troubleshooting.

### CI-019 — Reusable workflows adoption

- **Dependencies:** CI-003
- **Deliverables:** Convert `ci-frontend-sample.yml` and `ci-backend-sample.yml` to call `reusable-build-push.yml` with inputs.
- **Acceptance:** Reusable flows build/push images for both sample apps.

### CI-020 — Publish SHA tags explicitly

- **Dependencies:** CI-002
- **Deliverables:** Ensure `docker/metadata-action` config includes SHA tags (e.g., `type=sha,format=long`) so immutable tags like `c6cd88ef7de43520f6e95fcb87e6f2452b45edd4` are always available.
- **Acceptance:** New releases include a long SHA tag visible in GHCR; helm values can pin by SHA.
