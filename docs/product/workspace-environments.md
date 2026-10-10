---
title: Workspace environments
description: Understand the isolated workspace each run uses and how its services are defined.
group: Runtime
order: 40
---
## One run, one workspace

Each run works in its own isolated workspace on a worker. Only one run that
changes code controls a workspace at a time, which keeps changes, test results
and failures attributable to one attempt. A workspace is replaceable
infrastructure: destroying it leaves the task, run, decisions, changes,
validation and evidence intact.

## Environment definition

By default, a run starts in a reviewed minimal workspace: one non-root
container built on your server during installation. When a repository needs
services, the operator defines Compose documents for it in the repository
runtime policy (`BR_REPOSITORY_RUNTIME_POLICIES` in the API environment). The
task page shows the resolved environment read-only before the run starts, and
the start is refused if it changed after you reviewed it. **Override Compose
documents** lets you paste documents for one run.

BranchRunner does not read the repository's own Compose files automatically.
Every environment passes the same provisioning policy and capacity checks.

## Cleanup and evidence

Workspace deletion is separate from evidence retention. Built-in retention
deletes expired run artifacts that are not on hold; see
[backup and recovery](/docs/backup-and-recovery).
