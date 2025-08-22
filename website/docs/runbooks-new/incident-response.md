---
id: incident-response
slug: /runbooks/incident-response
sidebar_position: 19
---

# Incident Response Plan

:::info This document outlines our incident response plan, which is designed to help us respond to and resolve incidents in a timely and effective manner. :::

## Roles and Responsibilities

- **On-Call Engineer:** The first responder to an incident. Responsible for acknowledging the incident, assessing the impact, and initiating the response.
- **Incident Commander:** The person responsible for managing the incident response. This may be the on-call engineer or another designated person.
- **Team Members:** Responsible for assisting the incident commander and on-call engineer as needed.

## Severity Levels

| Severity | Description |
| --- | --- |
| **SEV 1** | **Critical issue affecting all users.** Examples: website down, data loss. |
| **SEV 2** | **Major issue affecting a subset of users.** Examples: a key feature is not working. |
| **SEV 3** | **Minor issue.** Examples: a cosmetic bug, a slow API endpoint. |

## Incident Response Checklist

1.  **Identify the incident:** The on-call engineer is alerted to an issue via our monitoring and alerting systems.
2.  **Acknowledge the incident:** The on-call engineer acknowledges the alert to let the team know they are responding.
3.  **Assess the impact:** The on-call engineer assesses the impact of the issue and assigns a severity level.
4.  **Communicate:** The on-call engineer communicates the issue to the team and stakeholders using the communications template below.
5.  **Mitigate:** The on-call engineer works to mitigate the issue and restore service as quickly as possible.
6.  **Resolve:** The on-call engineer resolves the issue and verifies that the fix is working.
7.  **Post-mortem:** The team conducts a post-mortem to identify the root cause of the issue and prevent it from happening again.

## Communications Template

```markdown title="Incident Communication Template"
**Title**: [SEV X] Brief description of the issue

**Status**: Investigating | Mitigating | Resolved

**Impact**: What is the impact of the issue?

**Next Steps**: What are the next steps?
```

## Post-mortem

A post-mortem is a meeting where the team discusses an incident after it has been resolved. The goal of a post-mortem is to identify the root cause of the incident and to create a plan to prevent it from happening again.

### Post-mortem Template

- **Title:** Post-mortem for [SEV X] Brief description of the issue
- **Date:**
- **Attendees:**
- **Timeline:** A detailed timeline of the incident.
- **Root Cause:** A detailed analysis of the root cause of the incident.
- **Action Items:** A list of action items to prevent the incident from happening again.
