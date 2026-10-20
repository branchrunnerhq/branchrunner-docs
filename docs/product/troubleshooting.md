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
  with `adapter_retired`.
- **Codex subscription not placed on a worker on another host:** that host
  needs its own remote selection and the owner's admission. The worker
  installer selects remote execution on the worker host when the control plane
  already had protected Codex selected. If it did not yet, if the host enrolled
  later with `br-worker-enrol`, or if `none` was recorded on the worker host,
  run this on the worker host once the control plane is ready:

  ```bash
  sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh codex-protected --control-plane
  ```

  Then the installation owner chooses **Admit for subscription execution** under
  **Runtime workers** after admitting the worker's runtime identity, and the
  page shows `admitted` or the reason it was refused. Without both, a Codex
  subscription run is not placed on that worker, or is refused with
  `remote_execution_tunnel_unavailable`. See
  [protected Codex on another host](/docs/runtime-workers#protected-codex-on-another-host).
- **Claude subscription not placed on a worker on another host:** this is
  expected in 1.0.0 (`claude_remote_execution_unavailable`). Claude
  subscription stages run only on the worker on the control-plane host.
- **Claude subscription needs attention:** its one-year token expired, or the
  organization's credentials were revoked after it signed in. **Verify** is not
  offered and runs cannot use it. For an expired token, choose **Sign in
  again** on the connection; the connection and its agent profiles stay. After
  a revocation, **Sign in again** is not offered: choose **Remove sign-in**, add
  a new Claude subscription connection, sign in on it and select it in the
  agent profiles that used the old one. See
  [Sign in again](/docs/agent-providers#sign-in-again).
- **Claude subscription signed in but not Execution available:** the card
  names the first thing to fix, and its details list each one with its action
  under **Agent execution**. See
  [when a Claude subscription can run agents](/docs/agent-providers#when-a-claude-subscription-can-run-agents).
- **Sign in again refused:** "Claude subscription operations on this
  connection are still running. Sign in again after they settle." Wait until
  the Claude stages that use the connection have finished or stopped, then try
  again.
- **Claude stage waits on the run page:** each Claude subscription stage, and
  each of the two Claude Review passes, starts only after its session approval.
  The person who started the run requests it under **Claude subscription ·
  session approval**, and an owner or administrator who signed in with their
  authenticator within the last five minutes approves it. If the decision
  deadline passed before the session started, choose **Request fresh Claude
  session approval**. See
  [Claude session approval](/docs/agent-providers#claude-session-approval).
- **Claude session approval request refused:** only runs that the connection's
  signing administrator starts can use it. That is the administrator who
  signed it in most recently, including with **Sign in again**. Have them start
  the run. A provider administrator who wants to start runs with the
  connection can choose **Sign in again** to become its signing administrator.
  The request is also refused while the run's workspace has no current lease on
  a healthy worker.
- **Claude stage failed to start:** a session start that was refused before
  any session ran fails the stage with a reason that starts with
  `workflow_claude_start_failed_`. The worker journal names the same cause.
  Retry the stage under **Retry stopped stages** on the run page. A reason
  ending in `claude_session_refused_native_session_mismatch` means the Claude
  native service refused the request; its journal
  (`journalctl -u branchrunner-claude-subscription-native.service`) names the
  check in a `claude_native_attest_refused` line. If that check is
  `claude_native_attest_repository_unregistered`, the repository was not yet
  registered for Claude; see the source keeper under
  [when a Claude subscription can run agents](/docs/agent-providers#when-a-claude-subscription-can-run-agents).
- **Run preflight shows Starting:** after a deploy, restart or upgrade, the
  protected Codex and Claude rows read **Starting** with a named state until
  their runtimes have attested and published again. They turn **Ready** by
  themselves. See [run preflight](/docs/readiness-reasons#run-preflight).
- **Upgrade refused with "The worker stopped without a clean drain":** the
  worker did not settle everything when it stopped. Its journal
  (`journalctl -u branchrunner-worker.service`) names each cause before the
  line `worker main finished (exit 75)`. Resolve it, then rerun the installer.
- **Validation failed:** reproduce the exact command inside the preserved
  workspace. If only the hygiene check ran, the repository has no supported npm
  suite; see [validation and approvals](/docs/validation-and-approvals).
- **Delivery blocked:** the repository's **Draft pull requests** panel lists
  what blocks publication, such as a missing GitHub App permission or a base
  branch the policy does not allow. The workspace and evidence remain available.
