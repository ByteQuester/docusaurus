---
id: 38-docs-refactor
slug: /tasks/50-docs-management/refactor
sidebar_position: 2
---

# Docs Refactor

:::info Goal Organize docs and tasks into clear, repeatable chunks to speed development and onboarding. Implement a stable taxonomy, move files accordingly, update indexes/links, and add category metadata for navigation. :::

<details>
<summary>TASK-001 — Define taxonomy and target structure</summary>

- **Status**: In Progress

**Target directories under `docs/docs/`**

- guides/: How-to flows (step-by-step), e.g., zero-to-hero, fresh start, onboarding sequences
- reference/: Concepts and components, e.g., security, network-policies, observability plan
- runbooks/: Operational procedures for incidents and routine ops
- integrations/: External systems (DNS, TLS, IRSA, registry)
- projects/: Portfolio and validation checklists

**Initial mapping (examples)**

- guides/
  - zero-to-hero-guided-path.md
  - fresh-start-redeploy.md
- reference/
  - security.md, network-policies.md, observability.md, scaling.md
  - kyverno-baseline-install.md, pod-security-admission.md
- runbooks/
  - tls-troubleshooting.md, argo-troubleshooting-\*.md, incident-response.md, known-issues.md
- integrations/
  - dns-setup.md, dns-tls.md, irsa-externaldns.md, registry-auth.md, sops-age.md
- projects/
  - portfolio.md, integration-links.md, verification-checklist.md

**Acceptance**

- Taxonomy agreed and recorded; next subtasks can execute file moves.

</details>

<details>
<summary>TASK-002 — Move Guides</summary>

- **Status**: Planned

**Moves**

- docs/docs/zero-to-hero-guided-path.md → docs/docs/guides/zero-to-hero-guided-path.md
- docs/docs/fresh-start-redeploy.md → docs/docs/guides/fresh-start-redeploy.md

**Category file**

- Create `docs/docs/guides/_category_.json` with label "Guides" and position 1.

**Acceptance**

- Files moved; imports/links updated in DOCS-LINKS-001; category renders in sidebar.

</details>

<details>
<summary>TASK-003 — Move Reference</summary>

- **Status**: Planned

**Moves (non-exhaustive, adjust during execution)**

- docs/docs/security.md → docs/docs/reference/security.md
- docs/docs/network-policies.md → docs/docs/reference/network-policies.md
- docs/docs/observability.md → docs/docs/reference/observability.md
- docs/docs/scaling.md → docs/docs/reference/scaling.md
- docs/docs/kyverno-baseline-install.md → docs/docs/reference/kyverno-baseline-install.md
- docs/docs/pod-security-admission.md → docs/docs/reference/pod-security-admission.md

**Category file**

- Create `docs/docs/reference/_category_.json` with label "Reference" and position 2.

**Acceptance**

- Files moved; imports/links updated in DOCS-LINKS-001; category renders in sidebar.

</details>

<details>
<summary>TASK-004 — Move Runbooks</summary>

- **Status**: Planned

**Moves**

- docs/docs/tls-troubleshooting.md → docs/docs/runbooks/tls-troubleshooting.md
- docs/docs/argo-troubleshooting-sync-permissions.md → docs/docs/runbooks/argo-troubleshooting-sync-permissions.md
- docs/docs/argo-troubleshooting-case-study.md → docs/docs/runbooks/argo-troubleshooting-case-study.md
- docs/docs/incident-response.md → docs/docs/runbooks/incident-response.md
- docs/docs/known-issues.md → docs/docs/runbooks/known-issues.md

**Category file**

- Create `docs/docs/runbooks/_category_.json` with label "Runbooks" and position 3.

**Acceptance**

- Files moved; imports/links updated in DOCS-LINKS-001; category renders in sidebar.

</details>

<details>
<summary>TASK-005 — Move Integrations</summary>

- **Status**: Planned

**Moves**

- docs/docs/dns-setup.md → docs/docs/integrations/dns-setup.md
- docs/docs/dns-tls.md → docs/docs/integrations/dns-tls.md
- docs/docs/irsa-externaldns.md → docs/docs/integrations/irsa-externaldns.md
- docs/docs/registry-auth.md → docs/docs/integrations/registry-auth.md
- docs/docs/sops-age.md → docs/docs/integrations/sops-age.md

**Category file**

- Create `docs/docs/integrations/_category_.json` with label "Integrations" and position 4.

**Acceptance**

- Files moved; imports/links updated in DOCS-LINKS-001; category renders in sidebar.

</details>

<details>
<summary>TASK-006 — Move Projects/Portfolio</summary>

- **Status**: Planned

**Moves**

- docs/docs/portfolio.md → docs/docs/projects/portfolio.md
- docs/docs/integration-links.md → docs/docs/projects/integration-links.md
- docs/docs/verification-checklist.md → docs/docs/projects/verification-checklist.md

**Category file**

- Create `docs/docs/projects/_category_.json` with label "Projects" and position 5.

**Acceptance**

- Files moved; imports/links updated in DOCS-LINKS-001; category renders in sidebar.

</details>

<details>
<summary>TASK-007 — Update index pages and sidebars</summary>

- **Status**: Planned

**Steps**

- Update `docs/docs/docs-index.md` links after moves.
- Ensure `intro.md` and `docs-index.md` appear at the top.
- Add `_category_.json` files to new directories with appropriate labels/positions.

**Acceptance**

- Sidebar shows Guides, Reference, Runbooks, Integrations, Projects sections with expected pages.

</details>

<details>
<summary>TASK-008 — Update cross-links</summary>

- **Status**: Planned

**Steps**

- Grep for moved paths and update links to new locations.
  ```bash
  rg -n "\((dns-setup|dns-tls|kyverno-baseline-install|network-policies|observability|scaling|tls-troubleshooting|argo-troubleshooting|incident-response|known-issues|irsa-externaldns|registry-auth|sops-age|portfolio|integration-links|verification-checklist)\.md\)"
  ```
- Adjust relative paths after moves.

**Acceptance**

- No broken internal links in docs; local docs build renders with expected navigation.

</details>

<details>
<summary>TASK-009 — Link check and docs build</summary>

- **Status**: Planned

**Steps**

- Run local docs build (if available) and/or a link checker.
- Fix any broken links or sidebar issues.

**Acceptance**

- Docs site builds successfully; no broken internal links.

</details>

:::warning Notes

- Move docs in small batches; commit after each subtask.
- Some references may break temporarily; fix in DOCS-LINKS-001. :::
