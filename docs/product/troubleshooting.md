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
- **Agent execution refused:** on the control-plane host, the installation
  selects `codex-protected` automatically once the worker installer has
  attested the host, and `claude-protected` once it has prepared the Claude
  runtime, and upgrades keep both. The worker installer summary prints the
  outcome, and `/ready` reports the composed selection in its
  `protected_execution` check. If nothing is selected, check whether an owner
  recorded the opt-out with `branchrunner-configure-execution.sh none`, which
  automation never overrides; select the runtime again by hand with
  `codex-protected` or `claude-protected`, or rerun the worker installer after
  fixing a refused attestation. API-key connections need no execution
  selection. The direct `codex` and `claude` adapters are retired and refused
  with `adapter_retired`. For protected Codex on a worker on another host, the
  installation owner must also choose **Admit for subscription execution**
  under **Runtime workers** after admitting its runtime identity. Without it, a
  Codex subscription run is not placed on that worker, or is refused with
  `remote_execution_tunnel_unavailable`. See
  [protected Codex on another host](/docs/runtime-workers#protected-codex-on-another-host).
- **Validation failed:** reproduce the exact command inside the preserved
  workspace. If only the hygiene check ran, the repository has no supported npm
  suite; see [validation and approvals](/docs/validation-and-approvals).
- **Delivery blocked:** the repository's **Draft pull requests** panel lists
  what blocks publication, such as a missing GitHub App permission or a base
  branch the policy does not allow. The workspace and evidence remain available.
