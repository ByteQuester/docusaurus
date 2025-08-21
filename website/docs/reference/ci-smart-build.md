---
id: ci-smart-build
slug: /reference/ci-smart-build
sidebar_position: 4
---

# CI Smart Builds

:::info Our Continuous Integration (CI) pipeline is designed to be efficient and fast by only building and testing what has changed. This is particularly important in a monorepo where many independent applications and services coexist. :::

:::note The full workflow can be viewed at [`.github/workflows/ci.yml`](https://github.com/your-org/your-repo/blob/main/.github/workflows/ci.yml). :::

## 1. Change Detection

The pipeline starts by determining which applications under the `apps/` directory have been modified in a pull request or push. This is handled by the `determine-changes` job.

- It uses the `dorny/paths-filter` action to identify files that have been added or modified.
- A subsequent step processes this list to extract the unique application names (e.g., `frontend`, `backend-sample`).
- The result is a JSON array of changed app names, which is passed as an output to the next jobs.

```yaml title=".github/workflows/ci.yml"
jobs:
  determine-changes:
    runs-on: ubuntu-latest
    outputs:
      apps: ${{ steps.apps.outputs.apps }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Detect changed files (apps/**)
        id: filter
        uses: dorny/paths-filter@v3
        with:
          list-files: json
          filters: |
            apps:
              - added|modified: 'apps/**'

      - name: Determine changed apps
        id: apps
        env:
          FILES_JSON: ${{ steps.filter.outputs.apps_files }}
        run: |
          echo "$FILES_JSON" | jq -r '.[]' | awk -F/ '/^apps\//{print $2}' | sort -u | jq -R -s 'split("\n") | map(select(length>0))' > apps.json
          echo "apps=$(cat apps.json)" >> $GITHUB_OUTPUT
```

## 2. Dynamic Build Matrix

The `build` job uses the output from the previous step to create a dynamic build matrix.

- The `if: needs.determine-changes.outputs.apps != '[]'` condition ensures the job only runs if there are changes in at least one application.
- The `strategy.matrix.app` is populated with the application names. This spins up a parallel job for each changed application.
- Inside the job, `pnpm --filter ${{ matrix.app }}` commands are used to run tests, linting, and builds scoped to only that single application, avoiding unnecessary work.

```yaml title=".github/workflows/ci.yml"
build:
  needs: determine-changes
  if: needs.determine-changes.outputs.apps != '[]'
  runs-on: ubuntu-latest
  strategy:
    matrix: ${{ fromJson(needs.determine-changes.outputs.apps) }}

  steps:
    # ... setup steps ...
    - name: Test
      run: pnpm --filter ${{ matrix.app }} test

    - name: Build
      run: pnpm --filter ${{ matrix.app }} build
```

## 3. Decoupled Helm & Security Jobs

Other jobs for Helm chart validation and security scanning run after the build is successful.

### Helm Testing

The `helm-test` job lints and templates the Helm charts.

:::warning Currently, it is statically configured to validate the `frontend` chart. :::

```yaml title=".github/workflows/ci.yml"
helm-test:
  runs-on: ubuntu-latest
  needs: build

  steps:
    # ... setup steps ...
    - name: Lint Helm chart
      run: helm lint charts/frontend
```

### Security Scanning & Publishing

The `security_and_publish` job performs several critical tasks:

- **Scanners**: It runs Gitleaks for secrets, Checkov for IaC, and Trivy for container image vulnerabilities. Reports are uploaded as SARIF files for integration with GitHub Security.
- **Publishing**: It builds and pushes the Docker image for the `frontend` application.

:::warning This step is not currently dynamic and is hardcoded to the `frontend` app. :::

```yaml title=".github/workflows/ci.yml"
security_and_publish:
  needs: [build, helm-test]
  runs-on: ubuntu-latest
  permissions:
    # ...
  steps:
    - name: Run Gitleaks
      # ...
    - name: Run Checkov
      # ...
    - name: Build and push Docker image
      id: build
      uses: docker/build-push-action@v5
      with:
        context: .
        file: apps/frontend/Dockerfile
        push: ${{ startsWith(github.ref, 'refs/tags/v') || github.ref == 'refs/heads/main' }}
        # ...
    - name: Run Trivy vulnerability scanner
      # ...
```
