# DOCS-LINKS-001 — Update cross-links

- Status: Planned

## Steps

- Grep for moved paths and update links to new locations.

```bash
rg -n "\((dns-setup|dns-tls|kyverno-baseline-install|network-policies|observability|scaling|tls-troubleshooting|argo-troubleshooting|incident-response|known-issues|irsa-externaldns|registry-auth|sops-age|portfolio|integration-links|verification-checklist)\.md\)"
```

- Adjust relative paths after moves.

## Acceptance

- No broken internal links in docs; local docs build renders with expected navigation.
