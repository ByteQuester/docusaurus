---
id: registry-image-pull
slug: /tasks/40-gitops/registry-image-pull
sidebar_position: 3
---

# Registry Image Pull Recovery (Atomic Tasks)

Goal: Fix ErrImagePull/ImagePullBackOff for frontend by ensuring the correct GHCR image exists, auth is configured, and Helm values reference it properly. Document for repeatability.

## REG-001 — Identify correct image repo and tags (pick chart first)

- Dependencies: CI-002
- Steps:
  - Decide which chart you are deploying right now:
    - Sample: `charts/frontend-sample` → image repo typically `ghcr.io/bytequester/frontend-sample`
    - Main: `charts/frontend` → image repo `ghcr.io/bytequester/hosting-monorepo` (single-repo image)
  - Check packages in GitHub UI → Packages for the matching image repository.
  - Or from a machine with Docker:
    - `echo $GHCR_TOKEN | docker login ghcr.io -u ByteQuester --password-stdin`
    - For sample: `docker manifest inspect ghcr.io/bytequester/frontend-sample:latest | jq .schemaVersion`
    - For main: `docker manifest inspect ghcr.io/bytequester/hosting-monorepo:latest | jq .schemaVersion`
  - If manifest not found, list tags via `crane ls ghcr.io/bytequester/<repo>` (optional) or trigger a new CI push.
- Acceptance: You know the exact repo and tag that exists for the chosen chart (`:latest` or a specific SHA tag).

## REG-002 — Ensure image exists (trigger build/push if needed)

- Dependencies: CI-002
- Steps:
  - Confirm `.github/workflows/ci.yml` pushes on main/tags.
  - Make a trivial change under `apps/frontend` and merge to `main` (or dispatch the workflow) to push a fresh image.
  - Verify in Packages UI or with `docker manifest inspect`.
- Acceptance: Target tag exists in GHCR.

## REG-003 — Create/refresh registry pull secrets (namespace)

- Dependencies: none
- Steps:
  - Create docker-registry secret in `frontend-dev` with a PAT that has `read:packages` scope (and SSO enabled if org requires):
    - `kubectl create secret docker-registry ghcr-credentials -n frontend-dev --docker-server=ghcr.io --docker-username=ByteQuester --docker-password=<PAT> --kubeconfig $KUBECONFIG --dry-run=client -o yaml | kubectl apply -f -`
  - Repeat for `backend-dev` if needed.
- Acceptance: `kubectl get secret ghcr-credentials -n frontend-dev` exists.

## REG-004 — Wire imagePullSecrets to workloads

- Dependencies: REG-003
- Paths (choose one):
  - A) Via Helm values (preferred): set `.Values.imagePullSecrets` or `.Values.serviceAccount.imagePullSecrets` in chart values.
  - B) Patch default ServiceAccount in namespace (fallback): `kubectl patch sa default -n frontend-dev -p '{"imagePullSecrets":[{"name":"ghcr-credentials"}]}' --kubeconfig $KUBECONFIG`.
- Acceptance: `kubectl get sa default -n frontend-dev -o yaml` shows `imagePullSecrets` or rendered Deployment includes the pull secret.

## REG-005 — Update Helm values with correct image (match the chart in use)

- Dependencies: REG-001
- Steps:
  - If deploying sample chart (`charts/frontend-sample/values.yaml`):
    - Set `image.repository: ghcr.io/bytequester/frontend-sample`
    - Set `image.tag: <existing-tag>` (e.g., the SHA or `latest` for the sample image)
  - If deploying main chart (`charts/frontend/values-dev.yaml`):
    - Set `image.repository: ghcr.io/bytequester/hosting-monorepo`
    - Set `image.tag: <existing-tag>` (avoid `dev` if it doesn’t exist; use `latest` or SHA)
    - For a known-good run, you can pin to: `c6cd88ef7de43520f6e95fcb87e6f2452b45edd4`
  - Ensure repository/owner are lowercase for GHCR.
- Acceptance: `helm template` of the chosen chart renders the expected image reference.

## REG-006 — Redeploy and verify

- Dependencies: REG-004, REG-005
- Steps:
  - If using Helm directly: run `helm upgrade --install` for the chosen chart and namespace.
  - If using Argo CD: sync the `frontend` or `frontend-sample` Application.
  - Watch pods: `kubectl get pods -n frontend-dev -w --kubeconfig $KUBECONFIG`.
- Acceptance: Frontend pods become Ready; no ImagePullBackOff.

## REG-007 — Document registry auth and image mapping

- Dependencies: REG-006
- Deliverables: `docs/docs/registry-auth.md` capturing:
  - How CI names images (mapping to repo path)
  - How to list/verify GHCR tags
  - How to create namespace pull secrets and wire them
  - Case-sensitivity notes for GHCR (owner lowercase)
- Acceptance: Doc renders and is referenced from `docs/docs/ci-smart-build.md` or onboarding.

## REG-008 — Optional: make images public or grant access

- Dependencies: none
- Steps:
  - Consider making the GHCR package public for demo purposes, or ensure PAT used in cluster has `read:packages` and org SSO approval.
- Acceptance: Pulls succeed without privileged tokens (public) or with least-privilege PAT.
