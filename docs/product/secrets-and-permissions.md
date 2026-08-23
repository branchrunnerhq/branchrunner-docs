---
title: Secrets and permissions
description: Control credential references, runtime injection, and human approval boundaries.
group: Configuration
order: 70
---
## Secret references
BranchRunner workflows store references rather than plaintext credentials. Workers resolve values at runtime only when the step and role are authorized.

## Permission layers
- Repository access controls which source a run may read or update.
- Tool policy controls which operations each role may request.
- Network policy controls reachable services.
- Human approvals control sensitive transitions and actions.

## Rotation
Rotate the value in its owning secret system. Audit which workflows reference the secret before deletion to avoid unexplained run failures.
