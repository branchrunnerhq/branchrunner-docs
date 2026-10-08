---
title: Release status
description: Distinguish the initial self-hosted release scope from completed acceptance and publication.
group: Start here
order: 15
---

## Initial release scope

The initial offering is the source-available Community edition, operated on your
own infrastructure. It targets Ubuntu 24.04 LTS and 26.04 LTS, each on amd64 and
arm64, with Codex/OpenAI and Claude Code/Anthropic. Each agent has an API-key
and a subscription configuration, so the release covers 16 configurations. Each
one needs its own release acceptance, and none has passed it yet. Other coding
agents, platforms, Cloud, Enterprise services and the VS Code extension are
outside this initial offering.

## What the current candidate implements

The source implements the following. None of it is a release claim until live
acceptance passes.

- A single-host installer for the control panel. It sets up HTTPS, the API,
  the web UI and PostgreSQL, then runs browser owner setup with password and
  TOTP sign-in.
- Built-in signing and handover custody, created by default. No external Vault
  is required; Vault stays an optional integration you select explicitly.
  Signing stays inactive until the installation owner approves the tenant
  signing policy.
- Remote worker enrollment. You publish the worker channel to one source
  range, run the worker installer with the server address, and enter a
  single-use token at a hidden prompt. An administrator then admits the worker
  explicitly. See [runtime workers](/docs/runtime-workers).
- Codex API keys for OpenAI, OpenAI-compatible endpoints (for example
  DeepSeek) and Azure OpenAI. Through the API, a run can select one exact key
  revision. See
  [agent providers](/docs/agent-providers).
- The path from repository or issue to agent, validation and repair,
  independent review, human approval and a draft pull request. BranchRunner
  never merges or deploys. If you cancel a publication, the branch and draft
  pull request stay on the repository host for you to clean up.

## Known limitations

- **Live acceptance is pending** for every configuration, and also for remote
  workers, OpenAI-compatible and Azure targets, and publication cancellation.
- **Claude Code subscription is on hold.** It waits on an owner decision:
  either an official CLI-managed persistent home directory or written
  authorization from Anthropic. Claude Code with an API key is unaffected.
- **No public container images before a stable version.** The installer builds
  its images on your host. Agent execution is disabled by default
  (`BR_EXECUTION_ADAPTER=none`). Turning it on is still a release-operator step.
- **Supported hosts** are Ubuntu 24.04 and 26.04 LTS. The installer's
  Ubuntu 22.04 and Debian paths are not supported configurations.
- **Version and licence dates are provisional.** The release version and its
  licence Change Date are set once the first public release date is known. See
  [licensing](/docs/licensing).

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
