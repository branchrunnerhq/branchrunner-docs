---
title: Workspace environments
description: Define isolated checkouts and services for reproducible implementation and validation.
group: Runtime
order: 40
---
## One run, one workspace
Each run receives its own Git worktree and service environment. Parallel runs never share a checkout, which keeps changes, test results, and failures attributable to one attempt.

## Environment definition
Define setup commands, required services, health checks, exposed preview ports, and cleanup policy with the repository. Pin toolchains where repeatability matters.

## Cleanup and evidence
Workspace deletion is separate from evidence retention. Export required logs and test results before cleanup, and choose retention according to your deployment policy.
