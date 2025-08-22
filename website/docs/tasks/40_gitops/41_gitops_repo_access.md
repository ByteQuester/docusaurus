---
id: 41-gitops-repo-access
slug: /tasks/40-gitops/gitops-repo-access
sidebar_position: 2
---

# GitOps Repository Access

### GITOPS-RA-001 — Remove plaintext credentials from repo

- **Dependencies**: None
- **Deliverables**:
  - Delete `argo-repo-secret.yaml` and any other secret files with real credentials from the repository.
  - Rotate the exposed Personal Access Token (PAT) immediately in GitHub.
  - Add `infra/gitops/repos/` to house templates only.
  - Add `*.secret.yaml` to `.gitignore`.
- **Acceptance Criteria**:
  - No live secrets are present in the Git repository.
  - The PAT has been successfully rotated.

### GITOPS-RA-002 — Choose credential method

- **Dependencies**: None
- **Deliverables**:
  - Choose one of the following credential methods:
    - **Deploy Key (Recommended for single repo, read-only)**: Generate an SSH keypair and add the public key as a GitHub Deploy Key with read-only access.
    - **GitHub App (Recommended for multiple repos)**: Install a GitHub App and grant it repository read access.
    - **Personal Access Token (PAT)**: Use a PAT with `repo` read access.
- **Acceptance Criteria**:
  - The selected method is documented in `docs/docs/argo-quickstart.md`.

### GITOPS-RA-003 — Register repo in Argo CD

- **Dependencies**: `GITOPS-RA-002`
- **Deliverables**:
  - Create a secret in the `argocd` namespace for the chosen credential method (Deploy Key or PAT).
- **Acceptance Criteria**:
  - The repository secret is listed by `kubectl get secret -n argocd -l argocd.argoproj.io/secret-type=repository`.

### GITOPS-RA-004 — Ensure URL matches Application exactly

- **Dependencies**: `GITOPS-RA-003`
- **Deliverables**:
  - Confirm that `spec.source.repoURL` in `infra/gitops/apps/frontend.yaml` exactly matches the `url` in the repository secret.
- **Acceptance Criteria**:
  - The values match character-for-character.

### GITOPS-RA-005 — Verify Argo can auth to GitHub

- **Dependencies**: `GITOPS-RA-003`
- **Deliverables**:
  - Verify that Argo CD can authenticate with GitHub using the chosen method.
- **Acceptance Criteria**:
  - Argo CD repository server logs show a successful fetch.
  - The appropriate API test returns a 200 status code.

### GITOPS-RA-006 — Check for duplicate repo secrets

- **Dependencies**: `GITOPS-RA-003`
- **Deliverables**:
  - Check for and remove any stale or incorrect repository secrets.
- **Acceptance Criteria**:
  - Only one repository secret for the given URL remains.

### GITOPS-RA-007 — Network / TLS sanity

- **Dependencies**: None
- **Deliverables**:
  - Verify that the Argo CD repository server can reach `https://github.com`.
- **Acceptance Criteria**:
  - An HTTPS request from the repository server to GitHub is successful.

### GITOPS-RA-008 — Resync Application

- **Dependencies**: `GITOPS-RA-004` through `GITOPS-RA-007`
- **Deliverables**:
  - Trigger a hard refresh of the application in Argo CD.
- **Acceptance Criteria**:
  - The application reports as "Synced" and "Healthy".

### GITOPS-RA-009 — Hygiene: move templates; add docs

- **Dependencies**: `GITOPS-RA-001`
- **Deliverables**:
  - Move the secret manifest example to `infra/gitops/repos/web-hosting-repo-secret.template.yaml`.
  - Add a `.gitignore` entry for `infra/gitops/repos/*.secret.yaml`.
  - Update `docs/docs/argo-quickstart.md` with the chosen authentication method.
- **Acceptance Criteria**:
  - The repository contains only templates.
  - The documentation shows the correct procedure.

### GITOPS-RA-010 — Safer alternative: GitHub App (optional)

- **Dependencies**: None
- **Deliverables**:
  - Create a GitHub App with read-only permissions and install it on the repository.
  - Register the app in Argo CD by creating a secret with the necessary credentials.
- **Acceptance Criteria**:
  - The repository syncs using the GitHub App.
