---
title: Release status
description: What the BranchRunner 1.0.0 release candidate includes, its known limitations, and what release acceptance still has to prove.
group: Start here
order: 15
---

## Initial release scope

The initial offering is the source-available Community Edition, version 1.0.0,
operated on your own infrastructure. Version 1.0.0 is a release candidate. Its
source and release artifacts will be published together once release
acceptance passes. Until then, the only published installer is an earlier
pre-release that installs the control panel with agent execution disabled.

Supported targets are Ubuntu 24.04 LTS and 26.04 LTS, each on amd64 and arm64,
with two coding agents: Codex (OpenAI) and Claude Code (Anthropic). Each agent
can use an API key or a subscription sign-in, so the release covers 16
configurations. Each one needs its own release acceptance, and none has passed
it yet. Other coding agents, other platforms, Cloud, Enterprise services and
the VS Code extension are outside this release.

## What 1.0.0 includes

These features are implemented in the 1.0.0 source. Until release acceptance
passes for a configuration, treat them as candidate functionality, not a
guarantee.

- **One-command installation.** The server installer builds every image on your
  host, sets up HTTPS, the API, the web UI and PostgreSQL, and prints a
  single-use browser link for owner setup with password and TOTP sign-in. The
  worker installer adds a runtime worker on the same server or on another host.
  The full product test suite is opt-in (`--verify`), never part of a customer
  installation. See the [quickstart](/docs/quickstart).
- **Single-host and remote workers.** A worker can share the server or enroll
  from another host through a mutual-TLS channel with a single-use token. The
  worker installs from the same published release archive as the server, with
  a checksum-pinned one-line command, and needs no Git checkout. An owner or
  administrator admits every worker explicitly, together with the runtime
  identity it reports. A worker on another host runs Codex subscription and
  API-key stages; Claude subscription stages run only on the worker on the
  control-plane host. See [runtime workers](/docs/runtime-workers).
- **GitHub App setup in the app.** **Integrations > Connect GitHub** creates the
  installation's own GitHub App on your account. You choose its repositories on
  GitHub.
- **Provider connections.** A Codex subscription sign-in and a Claude
  subscription sign-in through the official Claude Code CLI, both run inside
  protected containers; API keys for OpenAI (including OpenAI-compatible
  endpoints and Azure OpenAI) and for Anthropic. Up to four runs can share one
  subscription connection. Saved API keys are sealed at rest, and a saved key
  or Claude sign-in can be removed from **Integrations**. See
  [agent providers](/docs/agent-providers).
- **Connection priority, plan usage and failover.** An agent profile orders up
  to six connections of one agent by drag and drop. Each stage uses the first
  eligible connection, based on measured plan-limit usage, and moves to the next
  one when a plan limit, rate limit, quota or login stops it.
- **Theoretical and effective cost.** Each run and connection shows what its
  tokens would cost at API list price and what the work actually cost under
  your subscription fee or API discount.
- **Six-stage workflows.** Plan → Implement → Validate → Independent review →
  Approval → Handoff, which opens a draft pull request. See
  [workflows](/docs/workflows).
- **Validation.** BranchRunner detects a repository's npm `build`, `typecheck`,
  `lint` and `test` scripts and runs them offline in a hardened validation
  container. See [validation and approvals](/docs/validation-and-approvals).
- **Independent review and signed evidence.** A separate reviewer, a
  subscription or an API-key connection, evaluates the change against the
  requirements, and the installation signs completion, evidence and review
  records with its own keys.
- **Draft pull requests through the GitHub App.** After a person approves the
  completion review, BranchRunner opens a draft pull request. It has no merge or
  deploy capability.
- **Built-in custody and retention.** Signing, handover and retention keys are
  created on your server; no external Vault is required. Automatic deletion of
  expired run artifacts that are not on hold is on by default. See
  [data retention, deletion and custody](/docs/data-retention-deletion-custody).

## Known limitations

- **Release acceptance is pending** for all 16 configurations, for workers on a
  separate host, for OpenAI-compatible and Azure targets, for concurrent
  subscription sessions and for publication cancellation.
- **The complete run to a draft pull request is still being proven live.** Each
  part of the path is implemented, but an end-to-end run from task to draft pull
  request on an installed host has not been completed yet.
- **The 1.0.0 installer is not published yet.** The published pre-release
  installer sets up the control panel with agent execution disabled. Do not
  substitute it for the 1.0.0 release.
- **Agent execution needs the owner's selection.** After installation, the owner
  selects protected Codex or protected Claude execution on the server. A saved
  login alone does not authorize a run.
- **Fresh installation time is not yet proven against the 5-minute target.**
  Measured upgrades on the development host took about 2 minutes for the
  control plane and about 2 minutes for the worker. A first installation also
  builds every image from source and takes longer.
- **Validation supports npm projects only.** It needs a `package-lock.json` and
  does not support npm workspaces. Other repositories get only the baseline
  repository-hygiene check, and completion names the missing evidence.
- **Every run needs two providers and a human reviewer.** Every change is
  reviewed as Critical in 1.0.0, so the reviewer must come from a different
  provider and model family than every Implement step, and a qualified person
  who did not write the change must complete a human technical review. An
  installation with connections from only one provider cannot complete review.
  See
  [review requirements for every run](/docs/validation-and-approvals#review-requirements-for-every-run).
- **Previews are not available in 1.0.0.** BranchRunner does not serve a
  preview of a run's running application in this release. The run console and
  workspace detail show previews as unavailable, and no preview link can be
  opened or shared.
- **Failover stays within one agent.** A connection list cannot mix Codex and
  Claude connections. A stage switches only at its start or after a failure
  that changed nothing in the workspace.
- **Cost figures can be upper bounds.** When a producer reports no cache detail,
  all input is priced at the uncached rate and the figure is marked. Claude
  reports plan usage only after a warning threshold, so a Claude subscription
  can show "not measured yet".
- **Agent profiles cannot use skills or MCP servers yet.** A profile that
  selects them cannot run. See [MCP and tools](/docs/mcp-and-tools).
- **Claude subscription runs only on the control-plane host.** In 1.0.0 a run
  that uses a Claude subscription connection is placed only on the worker on
  the control-plane host; a worker on another host is skipped with
  `claude_remote_execution_unavailable`. Workers on other hosts run Codex
  subscription and API-key stages.
- **Protected Codex on another host needs two owner steps.** After the worker
  is admitted, the owner admits the host for Codex custody on the control-plane
  host, then selects protected Codex on the worker host with the control
  plane's execution document. See
  [runtime workers](/docs/runtime-workers#protected-codex-on-another-host).
- **No public container images before a stable version.** The installer builds
  the BranchRunner images on your host and pulls only digest-pinned upstream
  base images such as Node.js, Ubuntu and PostgreSQL.
- **Supported hosts** are Ubuntu 24.04 and 26.04 LTS. The installer's Ubuntu
  22.04 and Debian code paths are not supported configurations.
- **Backups are yours.** Restoring onto a clean host has not been exercised as a
  complete live drill. Built-in retention reports backup expiry as unavailable.
  See [backup and recovery](/docs/backup-and-recovery).

## Execution acceptance

The release must demonstrate the complete repository or issue → coding agent →
validation and repair → independent review → human approval → draft pull
request path. Worker enrollment and admission, source import, pinned runtime
images, current provider authority, published workflow binding and run
progression must all succeed. A healthy web page, saved login, workflow draft,
fixture run or unit test is not evidence of that path.

Interrupted execution, revocation, reboot, worker replacement, reconciliation
and recovery also require acceptance without duplicate or unauthorized effects.

## Publication and operating responsibility

The 1.0.0 source, artifacts, checksums, versioned instructions and acceptance
record will be published together when approved. Use that record to determine
the supported configuration.

You own installation, updates, monitoring, credentials, backup and restore,
retention and deletion across your storage and backups. This self-hosted
release includes no BranchRunner-operated support service, SLA, backup service,
retention service or deletion service. Product security and authorization
checks still apply.

## Report a security issue

Email `security@branchrunner.com`, the contact listed in
[security.txt](https://branchrunner.com/.well-known/security.txt). Do not put
details of a finding in a public issue or pull request. See
[security](/docs/security).

See [quickstart](/docs/quickstart), [licensing](/docs/licensing) and
[runtime workers](/docs/runtime-workers).
