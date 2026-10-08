---
title: Release status
description: Distinguish the initial self-hosted release scope from completed acceptance and publication.
group: Start here
order: 15
---

## Initial release scope

The initial offering is the source-available Community edition, version 1.0.0,
operated on your own infrastructure. Version 1.0.0 is a release candidate: its
source is prepared, and its release acceptance has not passed yet. It targets
Ubuntu 24.04 LTS and 26.04 LTS, each on amd64 and arm64, with Codex/OpenAI and
Claude Code/Anthropic. Each agent has an API-key
and a subscription configuration, so the release covers 16 configurations. Each
one needs its own release acceptance, and none has passed it yet. Other coding
agents, platforms, Cloud, Enterprise services and the VS Code extension are
outside this initial offering.

## What the current candidate implements

The source implements the following. None of it is a release claim until live
acceptance passes.

- A single-host installer for the control panel. It detects the Ubuntu
  release and the amd64 or arm64 architecture, builds the BranchRunner images
  on the host, and sets up HTTPS, the API, the web UI and PostgreSQL. It then
  runs browser owner setup with password and TOTP sign-in.
- Built-in signing, handover and retention custody, created by the installer.
  No external Vault is required; Vault stays an optional integration you
  select explicitly.
  Signing stays inactive until the installation owner approves the tenant
  signing policy.
- Remote worker enrollment. You publish the worker channel to one source
  range, run the worker installer with the server address, and enter a
  single-use token at a hidden prompt. An administrator then admits the worker
  explicitly. See [runtime workers](/docs/runtime-workers).
- Five provider connections: a Codex subscription login; Codex API keys for
  OpenAI, for OpenAI-compatible endpoints (for example DeepSeek) and for Azure
  OpenAI; a Claude Code API key for Anthropic; and a Claude subscription
  account signed in through the official Claude Code CLI. A direct Codex run
  can be pinned to one exact API-key revision. See
  [agent providers](/docs/agent-providers).
- The path from repository or issue to agent, validation and repair,
  independent review, human approval and a draft pull request. BranchRunner
  never merges or deploys. If you cancel a publication, the branch and draft
  pull request stay on the repository host for you to clean up.

## Known limitations

- **Live acceptance is pending** for every configuration, and also for remote
  workers, OpenAI-compatible and Azure targets, and publication cancellation.
- **Claude Code subscription is off until the owner selects it.** Accounts
  sign in through the official Claude Code CLI. The installation owner turns on
  protected Claude execution on the host; like every configuration, it still
  needs live acceptance.
- **No public container images before a stable version.** The installer builds
  the BranchRunner images on your host; it pulls only upstream base images such
  as Node.js, Ubuntu and PostgreSQL from their public registries. Agent
  execution is off after installation (`BR_EXECUTION_ADAPTER=none`, and no
  protected Codex or Claude execution selected). The installation owner turns
  it on explicitly.
- **Supported hosts** are Ubuntu 24.04 and 26.04 LTS. The installer's
  Ubuntu 22.04 and Debian paths are not supported configurations.
- **The licence Change Date is provisional.** The licence files name version
  1.0.0. Its Change Date is set to three years after 1.0.0's first public
  release date when that date is known. See [licensing](/docs/licensing).

## Execution acceptance

The release must demonstrate the complete repository/issue → coding agent →
validation and repair → independent review → human approval → draft pull request
path. Worker enrollment and admission, source-transfer settlement, pinned runtime
images, current provider authority, published workflow binding, and run
progression must all succeed. A healthy web page, saved login, workflow draft,
fixture run, or unit test is not evidence of that complete path.

Interrupted execution, revocation, reboot, worker replacement, reconciliation,
and recovery also require acceptance without duplicate or unauthorized effects.
These are release requirements, not features removed from the agreed scope.

## Publication and operating responsibility

The complete initial release's source, artifacts, checksums, versioned
instructions, and acceptance record will be published together when approved.
Use that record to determine the supported configuration. Earlier pre-release
installers can expose a control panel with execution disabled; they do not
establish acceptance of the initial agent-execution release.

You own installation, updates, monitoring, credentials, backup and restore,
retention, and deletion across your storage and backups. This self-hosted release
includes no BranchRunner-operated support service, SLA, backup service, retention
service, or deletion service. Product security and authorization checks still apply.

## Report a security issue

Email `security@branchrunner.com`, the contact listed in
[security.txt](https://branchrunner.com/.well-known/security.txt). Do not put
details of a finding in a public issue or pull request.

See [quickstart](/docs/quickstart), [licensing](/docs/licensing), and
[runtime workers](/docs/runtime-workers).
