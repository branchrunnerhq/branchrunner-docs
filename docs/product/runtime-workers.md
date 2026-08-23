---
title: Runtime workers
description: Operate the execution hosts that provision isolated workspaces and run agent steps.
group: Runtime
order: 30
---
## Responsibility boundary
Runtime workers receive signed work instructions from the control plane. Repository checkouts, agent processes, build services, and validation commands run on workers rather than in the control plane.

## Readiness
A worker is ready only when its container runtime, workspace storage, network policy, and control-plane connection pass health checks. Capacity is reserved for the full run, including approval waits.

## Operations
- Keep workers disposable and replaceable.
- Separate pools when repositories require different network or trust boundaries.
- Send worker logs to your normal operations system.
- Drain a worker before maintenance so active work remains attributable.

