---
id: incident-response
slug: /runbooks/incident-response
sidebar_position: 19
---

# Incident Response

:::info This document outlines our incident response plan. :::

## Severity Levels

| Severity  | Description                              |
| --------- | ---------------------------------------- |
| **SEV 1** | Critical issue affecting all users.      |
| **SEV 2** | Major issue affecting a subset of users. |
| **SEV 3** | Minor issue.                             |

## Incident Checklist

1.  **Identify the incident**: The on-call engineer is alerted to an issue.
2.  **Acknowledge the incident**: The on-call engineer acknowledges the alert.
3.  **Assess the impact**: The on-call engineer assesses the impact of the issue and assigns a severity level.
4.  **Communicate**: The on-call engineer communicates the issue to the team and stakeholders.
5.  **Mitigate**: The on-call engineer works to mitigate the issue.
6.  **Resolve**: The on-call engineer resolves the issue.
7.  **Post-mortem**: The team conducts a post-mortem to identify the root cause of the issue and prevent it from happening again.

## Communications Template

```markdown title="Incident Communication Template"
**Title**: [SEV X] Brief description of the issue

**Status**: Investigating | Mitigating | Resolved

**Impact**: What is the impact of the issue?

**Next Steps**: What are the next steps?
```
