---
title: Troubleshooting
description: Diagnose installation, worker readiness, provider, validation, and delivery failures.
group: Operate
order: 150
---
## Start with preserved state

Read the run transition, worker logs, failed command and reference identifier
before retrying. A retry should not erase the first failure.

## Where to look

- **Installation:** each installer run writes a complete log and a per-phase
  timing table to `/var/log/branchrunner-install/<UTC time>.<id>/` on the server
  and `/var/log/branchrunner-worker-install/` on a worker host. A failed run
  prints the end of the failing log and its path; fix the cause and rerun the
  same command.
- **Readiness:** `/ready` on the API lists each dependency check with its
  status. `/health` only shows that the API process is alive.
- **Activation:** **Runtime workers** shows the post-setup activation and
  signing status with a reason code. On the server,
  `journalctl -u branchrunner-signing-activation.service` has the details.
- **First run:** Global Home's **Get to your first run** marks each step done,
  open or **Could not check** with the reason.
- **Run start:** the run-start check lists every blocking item with its exact
  reason.

## Common checks

- **Worker unavailable:** verify admission, runtime health, capacity and
  control-plane connectivity, and that the worker runs the same release as the
  control plane.
- **Provider unavailable:** the attention queue names the waiting stage and the
  earliest plan reset. Check the connection's plan usage and login under
  **Integrations**, or add another connection to the agent's priority list.
- **Agent execution refused:** confirm that the owner selected protected Codex
  or protected Claude execution on the server after the latest upgrade.
- **Validation failed:** reproduce the exact command inside the preserved
  workspace. If only the hygiene check ran, the repository has no supported npm
  suite; see [validation and approvals](/docs/validation-and-approvals).
- **Delivery blocked:** the repository's **Draft pull requests** panel lists
  what blocks publication, such as a missing GitHub App permission or a base
  branch the policy does not allow. The workspace and evidence remain available.
