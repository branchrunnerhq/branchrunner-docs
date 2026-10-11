---
title: Quickstart
description: Install a self-hosted BranchRunner server and worker, connect GitHub and coding agents from two providers, and run the first task to a draft pull request.
group: Start here
order: 20
---
## Before you start

BranchRunner 1.0.0 is a release candidate. Its installer, source archive and
checksums will be published together once release acceptance passes. The only
installer published today is an earlier pre-release that sets up the control
panel with agent execution disabled. See [release status](/docs/release-status).

You operate the installation yourself. BranchRunner provides no support, SLA,
backup, retention or deletion service for self-hosted installations. You own
infrastructure, updates, monitoring, backup and restore, and data lifecycle
policies.

You need:

- a clean Ubuntu 24.04 or 26.04 LTS server on amd64 or arm64 with at least 8 GB
  RAM, 2 CPU cores and 60 GB free disk (the images are built on the server);
- a public DNS name for that server, with TCP 80 and 443 reachable;
- an open SSH session to the server while the installer runs;
- a GitHub account or organization that owns the repositories to work on;
- coding agent access from **two** providers: a Codex subscription or OpenAI
  API key, **and** a Claude subscription or Anthropic API key. One provider is
  not enough. Every change is reviewed as Critical risk, and a Critical review
  needs a reviewer from a different provider and model family than the
  implementer;
- a person who can act as the human technical reviewer. Every run also needs a
  human technical review from a qualified person who did not write the change.

The installer detects the Ubuntu release and architecture and stops on an
unknown platform. It installs Docker, nginx, certificate renewal and firewall
rules itself, runs PostgreSQL in a container, and builds the BranchRunner images
on your server. It pulls only digest-pinned upstream base images such as
Node.js, Ubuntu and PostgreSQL.

## Install the server

Run the versioned one-line command published with the release on
branchrunner.com. It downloads the installer, checks its SHA-256 checksum, then
runs it with `sudo`. The installer verifies the matching source archive before
it uses it.

The installer asks for the public hostname, an administrator email and the
organization name. It prints numbered progress for its 12 phases, a timing
table at the end, and writes a complete log under
`/var/log/branchrunner-install/`. It is safe to rerun.

When the server is installed, it prints a single-use setup link that is valid
for two hours. Open it in a browser to create the owner account with a password
and an authenticator app (TOTP). After owner setup, the server finishes its own
activation within seconds: provider admission, protected runtime preparation
and evidence signing. **Runtime workers** shows the activation status.

The same command then installs the runtime worker on this server, so a
single-server installation needs no second command. The installer waits for
owner setup and activation, then runs the release's same-host worker installer
itself (see [Install a worker](#install-a-worker)).

Measured upgrades on the development host took about 2 minutes for the server
and about 2 minutes for the worker. A first installation also builds every image
from source and takes longer.

## Install a worker

Coding agents and validation run on a runtime worker. The worker installs from
the same published release as the server. It needs no Git checkout.

- **On the same server**, the server installer installs the worker itself.
  After owner setup it runs the release's same-host worker installer in the
  same terminal. The worker joins over the loopback channel. The worker
  installer stops and restarts the control panel containers that share its
  service account, so the control panel is briefly unavailable. If you ran the
  server installer with `--no-worker`, run the same-host command it printed
  under "Next steps" after owner setup. If it stopped before the worker step
  finished, it printed the command that resumes,
  `sudo bash /opt/branchrunner/current/scripts/install.sh worker`. **Runtime workers > Add a worker** also shows the same-host command
  under **On this server's own host**.
- **On another host**, first rerun the server installer once with
  `--remote-workers <CIDR>[,<CIDR>...]` after owner setup. The installer then
  prints the remote worker command: the published installer in `worker` mode
  with `--server`. **Runtime workers > Add a worker** shows it under **On
  another host**. Run it on the worker host.

A worker on another host runs Codex subscription and API-key stages. In 1.0.0,
Claude subscription stages run only on the worker on the control-plane host, so
install that worker as well if you use a Claude subscription. See
[runtime workers](/docs/runtime-workers).

The worker installer runs fast installation checks, not the product test suite.
`--verify` adds the full suite, which takes about two hours and is meant for
developers and release candidates. When asked, paste a single-use token from
**Runtime workers > Add a worker** at the hidden prompt. The worker then shows
as **Awaiting admission**. Review the reported host and runtime identity, then
admit it; nothing is scheduled before that.

## Select agent execution

Agent execution stays off after installation. Select the protected runtime for
the agent you will use, on the server:

```bash
sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh codex-protected
# or, for a Claude subscription:
sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh claude-protected
```

The selection succeeds only when the locally built images and the host
attestation verify. After an upgrade, a selected protected Codex runtime is
selected again automatically once the worker installer has attested the host.
Protected Claude must be selected again after the worker installer has prepared
it for the new build.

For protected Codex on a worker on another host, two more owner steps follow
the worker's admission. On the control-plane host, admit that host for Codex
custody and print the control plane's execution document:

```bash
sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh codex-remote-host admit WORKER_ID HOST_KEY_SHA256 CLI_IMAGE SOURCE_IMAGE
sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh codex-remote-host document
```

Then, on the worker host, select protected Codex with that document:

```bash
sudo bash /opt/branchrunner/current/infra/deploy/branchrunner-configure-execution.sh codex-protected --control-plane DOCUMENT --worker-id WORKER_ID
```

[Runtime workers](/docs/runtime-workers#protected-codex-on-another-host)
explains where each value comes from.

## Before your first run

A run needs two providers and a human reviewer. Every change is classified
Critical risk until it has been classified in detail, so every run's
Independent review stage needs a reviewer from a different provider and model
family than the implementer, plus a human technical review. An installation
with a single provider can never pass Review.

Global Home shows **Get to your first run** with the same steps, each linked to
the page that completes it. A step reads as done only when the system that owns
it says so. Steps the page cannot read are marked **Confirm yourself**.

1. **Install the GitHub App.** Open **Integrations > Connect GitHub**. It creates
   this installation's own GitHub App on your account. Install it on the
   repositories BranchRunner may work on; that choice is made on GitHub.
2. **Connect and attach a repository.**
3. **Import the repository source** from the repository page.
4. **Admit a worker** under **Runtime workers**.
5. **Connect a coding agent provider** under **Integrations**: sign in to a Codex
   subscription with a device code, sign in to a Claude subscription through the
   official Claude Code CLI, or add an API key. See
   [agent providers](/docs/agent-providers).
6. **Publish an agent profile** under **Coding agents** for the implementer.
   Order its provider connections by priority.
7. **Connect a second provider for Review** under **Integrations**. If the
   implementer uses Codex or OpenAI, connect a Claude subscription or Anthropic
   API key, and the other way round. It must report agent execution as
   available.
8. **Publish a reviewer profile on that connection** under **Coding agents**: a
   profile with the reviewer role, on the second provider's connection.
9. **Approve the independent-review signing policy** under **Runtime workers**.
   The installer approves review and evidence signing by default, but its
   default review policy need not use a reviewer from the second provider. The
   installation owner chooses the second provider's reviewer and approves that
   policy. The checklist reads this step as done only when review signing is
   active and its reviewers can review a Critical change.
10. **Qualify a human technical reviewer.** An owner or administrator, signed
    in again with their authenticator, qualifies a reviewer under **Qualify a
    reviewer** in the run's completion review. The reviewer must not have
    written any of the change. A qualification lasts up to 90 days. This
    happens during the first run, so have the reviewer ready.
11. **Enforce protected credentials** under **Integrations**. Credential
    enforcement starts in shadow mode, and API-key model calls are refused
    until an owner or administrator turns it on. It cannot be turned off.
12. **Protected execution selected.** Today this is the root command in
    [Select agent execution](#select-agent-execution):
    `branchrunner-configure-execution.sh codex-protected` or `claude-protected`.
    The checklist reads it as done once a subscription connection reports agent
    execution.
13. **Publish a workflow.** The starter template contains the six stages: Plan,
    Implement, Validate, Independent review, Approval and Handoff.
14. **Create a task** with numbered requirements, or import a GitHub issue.

A run that uses an API-key connection also needs a credential approval for
that run, in the run console. By default one person gives it.

Then start a run from the task. The run-start check lists anything that still
blocks it, with the exact reason. Approve the plan, follow the run in its
console, and open the completion review. When you approve publication there,
BranchRunner opens a draft pull request through the GitHub App. It never merges
or deploys.

Next, read [workflows](/docs/workflows) and
[validation and approvals](/docs/validation-and-approvals).
