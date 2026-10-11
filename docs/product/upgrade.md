---
title: Upgrade
description: Upgrade the control plane and workers while preserving identity, evidence and compatibility.
group: Operate
order: 130
---
## Before upgrading

Read the release notes and the
[changelog](https://branchrunner.com/changelog), and take a complete
recovery set (see [backup and recovery](/docs/backup-and-recovery)). Database
migrations are forward-only: after a migration has committed, the only way back
is restoring that recovery set.

Earlier pre-release installations (`0.1.0-pre.*`) are not upgraded to 1.0.0.
Install 1.0.0 on a clean host instead.

## Upgrade sequence

1. Take a recovery set and check that it can be read.
2. Rerun the server installer for the new version. It keeps the installation
   identity, Worker CA, database volume, passwords and keys, and keeps the
   previous site serving while images build. It applies all new migrations in
   one transaction: if any of them fails, none is applied, the schema stays
   unchanged and the previous release keeps serving.
3. Run the new release's worker command on every worker host. The server
   installer prints it, and **Runtime workers > Add a worker** shows it. The
   worker channel refuses a control plane and worker with different protocol
   versions.
4. Check **Runtime workers** for activation and signing status. On the
   control-plane host, a runtime identity whose only change is images built
   from the new release is admitted automatically. If a worker reports any
   other changed runtime identity, for example after a new Docker version or
   security profile, or after any change on a worker on another host, review it
   and choose **Admit runtime identity**; until then source import and agent
   runs are refused on that worker.
5. Run a known workflow and inspect its evidence.

An upgrade re-prepares the protected Codex and Claude runtimes for the new
build and withdraws execution until the worker installer has attested and
prepared them; both selections are then restored automatically. A recorded
opt-out (`none`) stays in force. When a change affects a signed fact, such as the reviewer image, the next
activation prepares a successor policy; with owner approval mode, approve it
under **Runtime workers**.

Breaking releases include explicit migration notes in the release notes.
