---
title: Release status
description: What the BranchRunner 1.0.0 release covers, which configurations were verified live, its known limitations, and what the operator is responsible for.
group: Start here
order: 15
---

## Release scope

BranchRunner Community Edition 1.0.0 is the first public release of the
source-available, self-hosted product, operated on your own infrastructure.
The release consists of the installer `install.sh`, the source archive with its
`.sha256` file, `SOURCE_COMMIT`, `RELEASE.json`, `SHA256SUMS` and the release
notes. `RELEASE.json` and `SHA256SUMS` identify the exact source commit and the
bytes of every file. Cloud, Enterprise services and the VS Code extension are
not part of this release.

## Supported configurations

BranchRunner 1.0.0 supports exactly three platforms: Ubuntu 24.04 LTS, Ubuntu
26.04 LTS and Debian 13, on amd64 (x86_64).

| Platform | Status |
| --- | --- |
| Ubuntu 26.04 LTS, amd64 | Supported. Release acceptance, recorded before 1.0.0 is published: a clean installation with the published installer, a complete run with a Codex subscription implementer and a Claude subscription reviewer, a Codex API-key run and a Claude API-key run |
| Ubuntu 24.04 LTS, amd64 | Supported. Release acceptance, recorded before 1.0.0 is published: a clean installation with the published installer and a complete run |
| Debian 13 (trixie), amd64 | Supported. Release acceptance, recorded before 1.0.0 is published: a clean installation with the published installer |

Both installers accept only these three and refuse every other platform before
anything is installed, with one message that names them. Refused platforms
include Ubuntu 22.04 and interim Ubuntu releases, Debian 12 and other Debian
releases, arm64 and every other architecture, Arch Linux, AlmaLinux and
FreeBSD. arm64 is not supported in 1.0.0. Debian 12 is not supported because
its AppArmor 3.0 parser cannot read the worker's AppArmor profile pack, which
is written for AppArmor ABI 4.0. More platforms are planned for later releases.

The server needs at least 8 GB RAM and 2 CPU cores; the server installer
refuses a smaller host before it installs anything, because every run reserves
4 GiB and 2 CPUs next to the control plane. 16 GB RAM and 4 CPU cores are
recommended. Below 16 GB the installer builds the protected agent images two at a time
instead of four, and a host below the recommended size may hold only one run at
a time. Allow 60 GB of free disk: the protected agent images need 20 GB of it,
plus a build cache capped at 64 GB. Runs need about 90 GB more on each worker
host for [receiver storage](/docs/runtime-workers#receiver-storage).

The supported coding agents are Codex (OpenAI) and Claude Code (Anthropic).
Each can use an API key or a subscription sign-in. Every supported combination
that the table does not list as live-verified, such as a Claude subscription
implementer, a worker on a separate host, or an OpenAI-compatible or Azure
OpenAI target, is supported but not yet live-verified. A verified row on one
platform does not verify another. This page marks a row live-verified once its runs are recorded. Other coding agents
and model providers are not part of this release.

## What 1.0.0 includes

- **One-command installation.** The server installer builds every image on your
  host, sets up HTTPS, the API, the web UI and PostgreSQL, and prints a
  single-use browser link for owner setup with password and TOTP sign-in. After
  owner setup it installs a runtime worker on the same host itself. Installation
  never runs test suites: the image builds bundle the released code, and the
  host verification suite is opt-in (`--verify`). The installer prints a timing
  table for its 12 phases; the release notes publish the measured
  fresh-installation time and the host size it was measured on, rather than a
  fixed time target. See the [quickstart](/docs/quickstart).
- **Single-host and remote workers.** A worker can share the server or enroll
  from another host through a mutual-TLS channel with a single-use token. The
  worker installs from the same published release archive as the server, with
  a checksum-pinned one-line command, and needs no Git checkout. An owner or
  administrator admits every worker explicitly, together with the runtime
  identity it reports. On the control-plane host, an upgrade that changes only
  the images built from the release the control plane runs is admitted
  automatically; any other change, and every worker on a separate host, needs
  **Admit runtime identity**. A worker on another host runs Codex subscription
  and API-key stages; Claude subscription stages run only on the worker on the
  control-plane host. See [runtime workers](/docs/runtime-workers).
- **Protected execution selected automatically.** On the control-plane host,
  the installation selects protected Codex execution once the worker installer
  has attested the host, and protected Claude execution once it has prepared
  the Claude runtime. Upgrades keep the selection, and `none` is the recorded
  opt-out that automation never overrides. API-key connections need no
  execution selection. A saved login alone does not authorize a run.
- **Credential enforcement on from the start.** A fresh installation starts
  with protected credential enforcement on, so there is no activation step. It
  cannot be switched off.
- **GitHub App setup in the app.** **Integrations > Connect GitHub** creates the
  installation's own GitHub App on your account. You choose its repositories on
  GitHub.
- **Provider connections.** A Codex subscription signs in with a device code. A
  Claude subscription signs in through the official Claude Code CLI, which
  creates a token for the connection that is valid for one year. **Sign in
  again** renews the token on the same connection, and an installation can
  hold several Claude accounts, one per connection. Both subscriptions run
  inside protected containers. API keys work for OpenAI
  (including OpenAI-compatible endpoints and Azure OpenAI) and for Anthropic.
  Up to four runs can share one subscription connection. Saved API keys are
  sealed at rest, and a saved key or a subscription sign-in can be removed from
  **Integrations**. Admitted provider hosts, credential enforcement and API-key
  credential approval are under **Settings**. See
  [agent providers](/docs/agent-providers).
- **Connection priority, plan usage and failover.** An agent profile orders up
  to six connections of one agent by drag and drop. Each stage uses the first
  eligible connection, based on measured plan-limit usage, and moves to the next
  one when a plan limit, rate limit, quota or login stops it. Codex and Claude
  subscription cards show the plan windows the provider reported, such as the
  5-hour and weekly windows, at sign-in, at **Verify** and after each run.
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

- **Live verification covers the listed rows only.** Workers on a separate
  host, OpenAI-compatible and Azure targets, concurrent subscription sessions
  and publication cancellation are supported but not yet live-verified.
- **Three platforms, amd64 only.** arm64, Debian 12, Ubuntu 22.04 and every
  other platform are not supported in 1.0.0; the installers refuse them before
  anything is installed.
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
  all input is priced at the uncached rate and the figure is marked.
- **Agent profiles cannot use skills or MCP servers yet.** A profile that
  selects them cannot run. See [MCP and tools](/docs/mcp-and-tools).
- **Claude subscription runs on the control-plane worker only.** In 1.0.0 a run
  that uses a Claude subscription connection is placed only on the worker on
  the control-plane host; a worker on another host is skipped with
  `claude_remote_execution_unavailable`. The Claude token stays in custody on
  the control-plane host and is never delivered to another host. Workers on
  other hosts run Codex subscription and API-key stages.
- **Protected Codex on another host needs the owner's admission.** The worker
  installer on that host selects remote execution itself. After the worker and
  its runtime identity are admitted, the installation owner chooses **Admit for
  subscription execution** under **Runtime workers**. See
  [runtime workers](/docs/runtime-workers#protected-codex-on-another-host).
- **No public container images.** The installers build every BranchRunner image
  on your host from the released source and pull only digest-pinned upstream
  base images and PostgreSQL.
- **Backups are yours.** Restoring onto a clean host has not been exercised as a
  complete live drill. Built-in retention reports backup expiry as unavailable.
  See [backup and recovery](/docs/backup-and-recovery).

## What live verification covers

A configuration is listed as live-verified only after, on an installed host of
that operating system and architecture:

1. the installed source, version, image identities, migrations, TLS, owner
   setup and worker identity were checked; and
2. each run that the row names finished the complete repository or issue →
   agent → validation and repair → independent review → human approval →
   draft pull request path on that host, with its evidence retained.

Fixtures and local tests do not count as live verification. A healthy web page,
saved login, workflow draft or unit test is not evidence of that path.

## Operating responsibility

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
