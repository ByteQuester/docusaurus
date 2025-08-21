# NPOL-007 — Controlled egress to Internet

- Status: Completed
- Dependencies: NPOL-002

## Scope

Allow only required egress (DNS already allowed). Add per-namespace or per-app egress policies.

## Steps

We have reviewed the `frontend-sample` and `backend-sample` applications and have not identified any external dependencies that require egress traffic, other than DNS which is already allowed.

Therefore, no further egress policies are needed at this time.

## Acceptance

- No external dependencies have been identified, so no further egress policies are required.
