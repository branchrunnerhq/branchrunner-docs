---
title: Upgrade
description: Upgrade the control plane and workers while preserving evidence and compatibility.
group: Operate
order: 130
---
## Before upgrading
Read the changelog, back up the database and configuration, and verify that worker and control-plane versions are compatible. Drain active workers when a release changes runtime behavior.

## Upgrade sequence
1. Export a backup and test that it can be read.
2. Apply required database migrations once.
3. Upgrade the control plane.
4. Replace or restart worker pools in stages.
5. Run a known workflow and inspect its evidence.

Breaking releases include explicit migration notes in the [changelog](/changelog).

