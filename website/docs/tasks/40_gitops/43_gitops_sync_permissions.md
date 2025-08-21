# GitOps Sync Permissions (Atomic Tasks)

Goal: Resolve Argo CD `PermissionDenied` when syncing Applications and ensure the corrected `frontend-sample` deploys while removing stale configs.

## GITOPS-SYNC-001 — Inspect Argo CD state

- Dependencies: AWSZ-020
- Steps:
  - `kubectl -n argocd get pods`
  - `kubectl -n argocd get cm argocd-cm -o yaml | cat`
  - `kubectl -n argocd get cm argocd-rbac-cm -o yaml | cat`
  - `kubectl -n argocd get applications --all-namespaces | cat`
- Acceptance: You have a snapshot of config and see existing Applications.

## GITOPS-SYNC-002 — Ensure admin login is enabled

- Dependencies: GITOPS-SYNC-001
- Steps:
  - In `argocd-cm` (data): check `admin.enabled`. If present and set to "false", set to "true":
    - `kubectl -n argocd patch cm argocd-cm --type merge -p '{"data":{"admin.enabled":"true"}}'`
  - Restart server (optional): `kubectl -n argocd rollout restart deploy/argocd-server`
- Acceptance: Admin login permitted.

## GITOPS-SYNC-003 — RBAC policy grants

- Dependencies: GITOPS-SYNC-001
- Steps:
  - Edit `argocd-rbac-cm` to ensure the policy grants sync/manage:
    - `policy.csv` should include a broad admin policy, e.g.:
      - `p, role:admin, applications, *, *, allow`
      - `p, role:admin, projects, *, *, allow`
      - `g, admin, role:admin`
  - Apply via patch or `kubectl edit cm argocd-rbac-cm -n argocd`.
- Acceptance: RBAC policy contains admin role with application sync permissions.

## GITOPS-SYNC-004 — Login and generate token

- Dependencies: GITOPS-SYNC-002
- Steps:
  - `argocd login <argocd-host:port> --username admin --password <admin-password> --grpc-web`
  - `argocd account generate-token --account admin` (store securely)
- Acceptance: CLI authenticated; token generated.

## GITOPS-SYNC-005 — Clean up stale Applications

- Dependencies: GITOPS-003
- Steps:
  - Ensure only the intended Applications exist in `infra/gitops/apps/` (e.g., `frontend-sample.yaml`, `backend.yaml`, `root.yaml`).
  - Remove any obsolete `frontend` Application from the cluster:
    - `kubectl -n argocd delete application frontend || true`
- Acceptance: Only desired Applications remain.

## GITOPS-SYNC-006 — Apply corrected Applications

- Dependencies: GITOPS-003
- Steps:
  - Apply the updated app manifests:
    - `kubectl apply -f infra/gitops/apps/root.yaml`
    - `kubectl apply -f infra/gitops/apps/frontend-sample.yaml`
    - `kubectl apply -f infra/gitops/apps/backend.yaml` (if used)
- Acceptance: Applications exist with correct repoURL, path, and namespace (`argocd`).

## GITOPS-SYNC-007 — Trigger sync (two options)

- Dependencies: GITOPS-SYNC-006
- Option A (kubectl, avoids Argo RBAC on sync):
  - Annotate for hard refresh: `kubectl -n argocd annotate application frontend-sample argocd.argoproj.io/refresh=hard --overwrite`
- Option B (Argo CLI):
  - `argocd app sync frontend-sample`
- Acceptance: Sync is started.

## GITOPS-SYNC-008 — Verify health

- Dependencies: GITOPS-SYNC-007
- Steps:
  - `kubectl -n argocd get application frontend-sample -o yaml | yq '.status.sync.status,.status.health.status'`
  - Or via CLI: `argocd app get frontend-sample`
- Acceptance: Application is Synced/Healthy.

## GITOPS-SYNC-009 — Root app-of-apps (optional)

- Dependencies: AWSZ-023
- Steps:
  - Ensure `infra/gitops/apps/root.yaml` references `frontend-sample` and other child apps.
  - Sync the root: Option A annotate `root` Application; Option B `argocd app sync root`.
- Acceptance: Children managed under root in Argo.

## GITOPS-SYNC-010 — Document

- Dependencies: DOC-108
- Deliverables: Update `docs/docs/argo-quickstart.md` with:
  - Admin enablement, RBAC policy, login/token, repo registration (SSH deploy key), and sync methods (CLI vs annotate).
- Acceptance: Page renders; steps match what worked.

## GITOPS-SYNC-011 — Normalize Application namespaces to `argocd`

- Dependencies: GITOPS-SYNC-001
- Steps:
  - Ensure intended Applications exist in `infra/gitops/apps/`:
    - `root.yaml`, `frontend-sample.yaml`, `backend-sample.yaml`
  - In each file, set the Application namespace to `argocd`:
    - `metadata.namespace: argocd`
    - In `root.yaml`, also set `spec.destination.namespace: argocd`
  - Remove any existing Applications in `gitops` namespace (if present):
    - `kubectl -n gitops delete application root || true`
    - `kubectl -n gitops delete application frontend-sample || true`
    - `kubectl -n gitops delete application backend-sample || true`
  - Apply the corrected manifests:
    - `kubectl apply -f infra/gitops/apps/root.yaml`
    - `kubectl apply -f infra/gitops/apps/frontend-sample.yaml`
    - `kubectl apply -f infra/gitops/apps/backend-sample.yaml`
- Acceptance: Application objects exist in `argocd` namespace only; none remain in `gitops`.

## GITOPS-SYNC-012 — Trigger hard refresh and verify

- Dependencies: GITOPS-SYNC-011
- Steps:
  - Trigger a hard refresh (avoids Argo CLI RBAC):
    - `kubectl -n argocd annotate application root argocd.argoproj.io/refresh=hard --overwrite`
    - `kubectl -n argocd annotate application frontend-sample argocd.argoproj.io/refresh=hard --overwrite`
    - `kubectl -n argocd annotate application backend-sample argocd.argoproj.io/refresh=hard --overwrite`
  - Verify status via `kubectl` or Argo CLI:
    - `kubectl -n argocd get application frontend-sample -o yaml | yq '.status.sync.status,.status.health.status'`
    - `kubectl -n argocd get application backend-sample -o yaml | yq '.status.sync.status,.status.health.status'`
    - Optional: `argocd app get root`
- Acceptance: Apps in `argocd` show Synced/Healthy (or progressing to Healthy).

## GITOPS-SYNC-013 — Optional: enable managing Applications in non-`argocd` namespaces

- Dependencies: AWSZ-020
- Use this only if you intentionally want Application objects to live in namespaces like `gitops`.
- Steps:
  - Allow Argo CD to watch additional namespaces:
    - `kubectl -n argocd patch cm argocd-cm --type merge -p '{"data":{"application.namespaces":"gitops"}}'`
  - Restart Argo CD components to pick up changes:
    - `kubectl -n argocd rollout restart deployment/argocd-server`
    - `kubectl -n argocd rollout restart deployment/argocd-repo-server || true`
    - `kubectl -n argocd rollout restart statefulset/argocd-application-controller`
  - Ensure the project allows sources from these namespaces (update `default` or create a new project):
    - Save as `project-default-source-ns.yaml` and apply with `kubectl apply -f project-default-source-ns.yaml`:

```
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: default
  namespace: argocd
spec:
  sourceRepos:
    - '*'
  destinations:
    - namespace: '*'
      server: https://kubernetes.default.svc
  sourceNamespaces:
    - argocd
    - gitops
```

- Acceptance: Applications created in `gitops` are processed by the controller without PermissionDenied.

## GITOPS-SYNC-014 — RBAC: ensure admin sync permissions

- Dependencies: GITOPS-SYNC-001
- Steps:
  - Confirm `argocd-rbac-cm` includes admin role with application sync permissions:
    - Example policies:

```
p, role:admin, applications, *, *, allow
p, role:admin, projects, *, *, allow
g, admin, role:admin
```

- Apply via patch or `kubectl edit cm argocd-rbac-cm -n argocd`.
- Acceptance: Admin can list/get/sync Applications via CLI; no PermissionDenied.

## GITOPS-SYNC-015 — Repository secret hygiene (rotate and remove from git)

- Dependencies: SEC-OPS
- Steps:
  - Remove any committed private keys from the repository (rotate credentials in GitHub/SSH):
    - `git rm --cached infra/gitops/repos/web-hosting-repo-secret.yaml`
    - Rotate keys in your VCS/registry as appropriate
  - Recreate the Argo CD repo secret directly in the cluster:
    - `kubectl -n argocd delete secret web-hosting-repo-secret || true`
    - `kubectl -n argocd create secret generic web-hosting-repo-secret \ --from-literal=url=git@github.com:ByteQuester/hosting-monorepo.git \ --from-file=sshPrivateKey=/secure/path/to/id_ed25519`
    - `kubectl -n argocd label secret web-hosting-repo-secret argocd.argoproj.io/secret-type=repository --overwrite`
- Acceptance: No private keys stored in git; Argo can still access the repo.

## GITOPS-SYNC-016 — Last resort: recreate Applications from scratch

- Dependencies: GITOPS-SYNC-011
- Steps:
  - Delete existing Applications in `argocd`:
    - `kubectl -n argocd delete application root || true`
    - `kubectl -n argocd delete application frontend-sample || true`
    - `kubectl -n argocd delete application backend-sample || true`
  - Re-apply manifests:
    - `kubectl apply -f infra/gitops/apps/root.yaml`
    - `kubectl apply -f infra/gitops/apps/frontend-sample.yaml`
    - `kubectl apply -f infra/gitops/apps/backend-sample.yaml`
  - Trigger hard refresh per GITOPS-SYNC-012.
- Acceptance: Fresh Application objects created and managed successfully.

---

See the detailed runbook with command snippets and explanation:

- `docs/docs/argo-troubleshooting-sync-permissions.md`
