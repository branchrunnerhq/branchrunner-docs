---
title: Quickstart
description: Prepare a self-hosted BranchRunner deployment and run the first controlled task.
group: Start here
order: 20
---
## Before you start
The initial release is the self-hosted product for Ubuntu 24.04 and 26.04 LTS
on amd64 or arm64. More platforms are planned. Cloud and the VS Code extension
are outside this release. Source and release artifacts will be published when
ready; release validation is still in progress.

You operate the installation yourself. BranchRunner does not provide support or an
SLA, backups, retention services, or deletion services for self-hosted installations.
You own infrastructure, updates, monitoring, backup/restore, and data lifecycle policies.

You need:

- a clean Ubuntu 24.04 or 26.04 LTS server with at least 8 GB RAM, 2 CPU cores
  and 60 GB free disk (images are built on the host);
- a public DNS name for that server, with TCP 80 and 443 reachable;
- an open SSH session to the server while the installer runs;
- credentials for at least one supported agent provider, for later runs.

The installer installs Docker, PostgreSQL, nginx, certificate renewal and
firewall rules itself. It builds the BranchRunner images on your server; no
BranchRunner container image will be published to a public registry before a
stable version.

## Start the deployment
Use the versioned one-line command and checksum published with the release on
branchrunner.com. The installer asks for the public hostname, an administrator
email and the organization name. It prints a single-use browser setup link.
Open it to set your password and authenticator (TOTP).

Then verify TLS and owner setup. Before any run, also check the exact worker
identity, its current certificate and incarnation, administrative admission,
capabilities and runtime image pins. A heartbeat alone does not establish
execution readiness.

Published pre-release installers set up the control panel with agent execution
disabled (`BR_EXECUTION_ADAPTER=none`). The full initial release still needs
acceptance and publication. See [release status](/docs/release-status); do not
substitute an older installer for the accepted execution release.

## Add a worker
To run workers on other hosts, rerun the server installer with
`--remote-workers` and the source address range of your workers. On each worker
host, run the worker installer with `--server` and your server's address, then
enter a single-use token from **Runtime workers > Add a worker** when asked. See
[runtime workers](/docs/runtime-workers).

## Run a task
1. Connect a repository with read and write access appropriate to the workflow.
2. Configure a supported Codex or Claude provider and confirm current execution
   authority; saved credentials alone are insufficient.
3. Create a task with numbered requirements and an owner.
4. Publish and bind the workflow, settle repository source transfer, then start
   a run, review its proposed scope, and approve implementation.
5. Inspect evidence before approving delivery. Delivery creates a draft pull
   request only.

Next, read [workspace environments](/docs/workspace-environments) and [validation and approvals](/docs/validation-and-approvals).
