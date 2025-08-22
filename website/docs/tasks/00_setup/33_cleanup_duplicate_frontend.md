---
id: cleanup-duplicate-frontend
slug: /tasks/00-setup/cleanup-duplicate-frontend
sidebar_position: 3
---

# Cleanup Duplicate Frontend

:::info

**Goal:** Ensure single source of truth at `apps/frontend`.

:::

### CLEAN-001 — Inventory duplicate paths

- **Status:** In Progress
- **Dependencies:** REPO-002
- **Steps:**
  - Search for `env/dev/src/frontend/*` or similar legacy paths.
- **Acceptance:** Inventory recorded; duplicates removed or redirected.
