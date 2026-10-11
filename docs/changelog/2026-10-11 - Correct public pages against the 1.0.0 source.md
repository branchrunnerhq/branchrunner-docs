# Correct public pages against the 1.0.0 source

The product pages now match what the 1.0.0 release candidate source does for
separate-host workers, run console controls, webhooks, previews and bug
reports.

## Changed

- **Claude subscription placement.** Release status, quickstart, runtime
  workers, agent providers and the introduction state that in 1.0.0 Claude
  subscription stages run only on the worker on the control-plane host. A
  worker on another host runs Codex subscription and API-key stages; run
  placement skips it for a Claude subscription run with
  `claude_remote_execution_unavailable`.
- **Protected Codex on another host.** Quickstart and runtime workers document
  the two owner steps: `codex-remote-host admit` and `document` on the
  control-plane host, then
  `codex-protected --control-plane DOCUMENT --worker-id WORKER_ID` on the
  worker host, and where each value comes from.
- **Run console controls.** Tasks and runs no longer promise interrupt, redirect
  or cancel for every agent. You can redirect a running Codex subscription turn;
  other stages take no input, a Claude subscription result appears when its
  stage settles, a pause takes effect at the next stage boundary, and cancel
  stops the whole run.
- **Webhooks.** The GitHub App's `push` and `pull_request` webhooks keep
  branches and draft pull requests reconciled; repository grants are rechecked
  every few minutes rather than driven by webhooks.
- **Readiness reason codes.** There is no BranchRunner support service, so the
  page asks you to include a code or log line in a bug report and lists what
  that report should contain. The readiness runbook is cited as part of the
  product source.
- **Previews.** Data retention qualifies preview links and preview identities
  with "when the optional preview edge is configured"; the 1.0.0 installer
  does not configure one.
- **Worker commands.** Runtime workers shows the installer address as a
  `<release URL>` placeholder, because the 1.0.0 release is not published yet.
- The repository's ignore-file comment is neutral.

## Removed

- The "Separate-host workers need three image pins" limitation. Each worker's
  locally built images are part of its admitted runtime identity, and the
  control plane needs no image configuration.
- A workspace changelog entry from 2026-09-21 that did not describe the
  documentation.
