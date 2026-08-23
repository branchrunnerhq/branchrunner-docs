---
title: Backup and recovery
description: Protect control-plane state and verify that recovery works before it is needed.
group: Operate
order: 140
---
## What to protect
Back up the control-plane database, deployment configuration, encryption material, and references needed to reconnect external systems. Repository source remains authoritative in your Git host.

## Recovery practice
Restore into an isolated environment, verify users and configuration, inspect historical evidence, and run a non-production workflow. Record recovery time and any manual steps.

Workspace storage is disposable execution state and should not be the only copy of required evidence.
