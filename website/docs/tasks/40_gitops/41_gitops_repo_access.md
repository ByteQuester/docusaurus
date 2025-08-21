---
id: gitops-repo-access
slug: /tasks/40-gitops/gitops-repo-access
sidebar_position: 2
---

# GitOps Repository Access (Atomic Tasks)

Goal: Resolve Argo CD "repository not found" by correctly registering the Git repo with proper credentials and hygiene (no plaintext secrets in repo).

## GITOPS-RA-001 — Remove plaintext credentials from repo

- Dependencies: none
- Steps:
  - Delete `argo-repo-secret.yaml` and any other secret files with real credentials from the repo.
  - Rotate the exposed PAT immediately in GitHub (Settings → Developer settings → Personal access tokens), or prefer a new credential method (deploy key or GitHub App).
  - Add `infra/gitops/repos/` to house templates only; add `*.secret.yaml` to `.gitignore`.
- Acceptance: No live secrets in git; PAT rotated.

## GITOPS-RA-002 — Choose credential method

- Dependencies: none
- Options (pick one):
  - A) Deploy key (recommended for single repo, read-only): generate SSH keypair; add public key as GitHub Deploy Key on `mpo/web-hosting` with read-only.
  - B) GitHub App (recommended for org/multiple repos): install app and grant repo read; configure Argo accordingly.
  - C) PAT (acceptable for quick start): token must have `repo` read access and belong to a user with access to the private repo.
- Acceptance: Selected method documented in `docs/docs/argo-quickstart.md`.

## GITOPS-RA-003 — Register repo in Argo CD (do not commit secrets)

- Dependencies: GITOPS-RA-002
- Steps (pick based on method):
  - A) Deploy key (SSH):
    - Create secret in `argocd` namespace (out of band; do not commit):
      - kind: Secret
      - metadata.labels: `argocd.argoproj.io/secret-type: repository`
      - stringData:
        - url: `git@github.com:mpo/web-hosting.git`
        - sshPrivateKey: |- <private key>
  - C) PAT (HTTPS):
    - Secret (out of band):
      - stringData:
        - url: `https://github.com/mpo/web-hosting.git`
        - username: <github-username-with-access>
        - password: <personal-access-token>
  - Apply via kubectl (do not use sudo unless passing same kubeconfig): `kubectl apply -f <secret-file> -n argocd --kubeconfig $KUBECONFIG`
- Acceptance: `kubectl get secret -n argocd -l argocd.argoproj.io/secret-type=repository` lists your repo secret.

## GITOPS-RA-004 — Ensure URL matches Application exactly

- Dependencies: GITOPS-RA-003
- Steps:
  - Confirm `infra/gitops/apps/frontend.yaml` `spec.source.repoURL` exactly equals the `url` in the repo secret (including scheme and .git suffix if used).
- Acceptance: Values match character-for-character.

## GITOPS-RA-005 — Verify Argo can auth to GitHub

- Dependencies: GITOPS-RA-003
- Steps:
  - Get repo-server logs: `kubectl logs deploy/argocd-repo-server -n argocd --kubeconfig $KUBECONFIG | tail -200`
  - If using PAT, verify the PAT has access to the repo (from any shell): `curl -u USER:TOKEN https://api.github.com/repos/mpo/web-hosting | jq '.private,.name'` (expect 200 and correct repo).
  - If using SSH, test key locally: `GIT_SSH_COMMAND="ssh -i ./id_rsa -o StrictHostKeyChecking=no" git ls-remote git@github.com:mpo/web-hosting.git`.
- Acceptance: Logs show successful fetch or at least authentication success; API test returns 200.

## GITOPS-RA-006 — Check for duplicate repo secrets

- Dependencies: GITOPS-RA-003
- Steps:
  - `kubectl get secret -n argocd -l argocd.argoproj.io/secret-type=repository -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.data.url}{"\n"}{end}' --kubeconfig $KUBECONFIG | base64 -d`
  - If multiple secrets reference the same URL, remove stale/incorrect ones.
- Acceptance: Only one repo secret for the given URL remains.

## GITOPS-RA-007 — Network / TLS sanity

- Dependencies: none
- Steps:
  - From repo-server pod: `kubectl exec -n argocd deploy/argocd-repo-server -- curl -I https://github.com/mpo/web-hosting.git` (or use busybox curl)
  - If corporate proxy/egress, ensure repo server has outbound internet; configure proxies in Argo if needed.
- Acceptance: HTTPS reachable from repo server.

## GITOPS-RA-008 — Resync Application

- Dependencies: GITOPS-RA-004..GITOPS-RA-007
- Steps:
  - `kubectl annotate app frontend -n argocd argocd.argoproj.io/refresh=hard --overwrite --kubeconfig $KUBECONFIG` (or use UI Sync)
  - Check status: `kubectl get application frontend -n argocd -o yaml --kubeconfig $KUBECONFIG | yq '.status.sync.status,.status.health.status'`
- Acceptance: Application reports Synced/Healthy.

## GITOPS-RA-009 — Hygiene: move templates; add docs

- Dependencies: GITOPS-RA-001
- Steps:
  - Move the secret manifest example to `infra/gitops/repos/web-hosting-repo-secret.template.yaml` with placeholder values and comments warning not to commit real secrets.
  - Add `.gitignore` entry for `infra/gitops/repos/*.secret.yaml`.
  - Update `docs/docs/argo-quickstart.md` with the chosen auth method steps and verification commands.
- Acceptance: Repo contains only templates; docs show the correct procedure.

## GITOPS-RA-010 — Safer alternative: GitHub App (optional)

- Dependencies: none
- Steps:
  - Create a GitHub App with read-only contents permission; install on the repo.
  - Register in Argo by creating a secret with `githubAppID`, `githubAppInstallationID`, and `githubAppPrivateKey` fields per Argo docs.
- Acceptance: Repo syncs using GitHub App; no user PAT needed.
