# DOCS-PLAN-001 — Taxonomy and target structure

- Status: In Progress

## Target directories under `docs/docs/`

- guides/: How-to flows (step-by-step), e.g., zero-to-hero, fresh start, onboarding sequences
- reference/: Concepts and components, e.g., security, network-policies, observability plan
- runbooks/: Operational procedures for incidents and routine ops
- integrations/: External systems (DNS, TLS, IRSA, registry)
- projects/: Portfolio and validation checklists

## Initial mapping (examples)

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

## Acceptance

- Taxonomy agreed and recorded; next subtasks can execute file moves.
