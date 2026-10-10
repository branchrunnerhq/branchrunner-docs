---
title: Upgrade
description: Upgrade the control plane and workers while preserving identity, evidence and compatibility.
group: Operate
order: 130
---
## Before upgrading

Read the release notes and the [changelog](/changelog), and take a complete
recovery set (see [backup and recovery](/docs/backup-and-recovery)). Database
migrations are forward-only: after a migration has committed, the only way back
is restoring that recovery set.

Earlier pre-release installations (`0.1.0-pre.*`) are not upgraded to 1.0.0.
Install 1.0.0 on a clean host instead.

## Upgrade sequence

1. Take a recovery set and check that it can be read.
2. Rerun the server installer for the new version. It keeps the installation
   identity, Worker CA, database volume, passwords and keys, applies the new
   migrations once, and keeps the previous site serving while images build.
3. Rerun the worker installer on every worker host. The worker channel refuses a
   control plane and worker with different protocol versions.
4. Check **Runtime workers** for activation and signing status.
5. Run a known workflow and inspect its evidence.

An upgrade re-prepares a protected Codex runtime for the new build and
withdraws execution until the worker installer has attested the host; a
selection that existed before the upgrade is then restored automatically.
Protected Claude must be selected again after the worker installer has prepared
it. When a change affects a signed fact, such as the reviewer image, the next
activation prepares a successor policy; with owner approval mode, approve it
under **Runtime workers**.

Breaking releases include explicit migration notes in the release notes.
