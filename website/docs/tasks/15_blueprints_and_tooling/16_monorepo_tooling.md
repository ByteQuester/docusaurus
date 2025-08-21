---
id: monorepo-tooling-evaluation
slug: /tasks/15-blueprints-and-tooling/monorepo-tooling-evaluation
sidebar_position: 2
---

# Monorepo Tooling Evaluation

:::info **Goal:** Decide whether to adopt Turborepo, Nx, or Lerna on top of pnpm workspaces. Keep the current repo productive; adopt only if it adds clear value. :::

### MRP-001 — Baseline capture

- **Dependencies:** none
- **Deliverables:** Document current build/test times locally and in CI for `frontend`, `backend-sample`, and charts. Note cache hits/misses.
- **Acceptance:** `docs/docs/monorepo-baseline.md` contains timings and observations.

### MRP-002 — Turborepo POC (local)

- **Dependencies:** MRP-001
- **Deliverables:** Add `turbo.json` with pipelines (build, test, lint, typecheck) and per-app tasks; wire scripts to Turbo for local runs.
- **Acceptance:** `pnpm dlx turbo run build --filter=...` works; local build time improves or task orchestration is simpler.

### MRP-003 — Turborepo remote cache (optional)

- **Dependencies:** MRP-002
- **Deliverables:** Configure remote cache (Vercel Remote Cache or self-hosted); document secrets placement.
- **Acceptance:** CI and local share cache; repeat CI runs show cache hits.

### MRP-004 — Nx POC (local)

- **Dependencies:** MRP-001
- **Deliverables:** Add Nx minimal config leveraging `nx.json` and `package.json` scripts; enable `affected` graph for `apps/*`.
- **Acceptance:** `npx nx affected --target=build` builds only changed apps locally.

### MRP-005 — CI integration option A (Turbo)

- **Dependencies:** MRP-002
- **Deliverables:** Add a CI job that runs `turbo run` with `--remote-cache` if configured; keep existing matrix path as fallback.
- **Acceptance:** CI jobs use Turbo successfully; matrix fallback still works.

### MRP-006 — CI integration option B (Nx)

- **Dependencies:** MRP-004
- **Deliverables:** Add a CI job that uses `nx affected --target=build,test,lint` to detect changes; keep existing matrix path as fallback.
- **Acceptance:** CI jobs use Nx successfully; matrix fallback still works.

### MRP-007 — Decision record

- **Dependencies:** MRP-002, MRP-004
- **Deliverables:** Architecture Decision Record with pros/cons and decision (adopt Turbo/Nx or stay with pnpm-only for now).
- **Acceptance:** `docs/docs/adr/monorepo-tooling.md` published with rationale and roll-forward/rollback notes.

### MRP-008 — Defer adoption (if chosen)

- **Dependencies:** MRP-007
- **Deliverables:** If deferring, remove POC configs from main branch; keep ADR; open backlog tasks for future revisit.
- **Acceptance:** Repo remains pnpm-first; issues opened for future revisit.

### MRP-009 — Adopt selected tool (if chosen)

- **Dependencies:** MRP-007
- **Deliverables:** Keep chosen config; update developer docs; update CI to prefer selected tool; ensure parity with prior behavior.
- **Acceptance:** All builds/tests pass; developer onboarding doc updated.
