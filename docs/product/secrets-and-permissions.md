---
title: Secrets and permissions
description: Control credential references, runtime injection, and human approval boundaries.
group: Configuration
order: 70
---
## Secret references
BranchRunner workflows store references rather than plaintext credentials. Workers resolve values at runtime only when the step and role are authorized.

## Built-in key custody
The installer creates the signing, handover and retention keys on the server
itself, in root-owned configuration directories with restricted file modes. No
external Vault is required. Vault remains an optional integration that you
select explicitly. Include these keys in your backups; see
[backup and recovery](/docs/backup-and-recovery).

## Permission layers
- Repository access controls which source a run may read or update.
- Tool policy controls which operations each role may request.
- Network policy controls reachable services.
- Human approvals control sensitive transitions and actions.

## Rotation
Rotate the value in its owning secret system. Audit which workflows reference the secret before deletion to avoid unexplained run failures.
