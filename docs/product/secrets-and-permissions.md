---
title: Secrets and permissions
description: Control credential custody, runtime injection, and human approval boundaries.
group: Configuration
order: 70
---
## Credential custody

Provider API keys are held by the installation's provider service, and
subscription sign-ins by the protected Codex and Claude custody services, all
encrypted on your server. Each saved API key is encrypted with AES-256-GCM
under its own data key, which is wrapped with the provider service's enrollment
key, kept apart from the key files. API-key stages call the provider through the
installation's provider gateway, and subscription sign-ins are used only by the
official CLIs inside protected containers. A Claude subscription's token stays
on the control-plane host. GitHub
installation tokens stay inside the SCM service and are never returned to the
API, worker, agent, database or logs.

A fresh installation starts with protected credential enforcement on, so there
is no activation step, and it cannot be switched off. An installation upgraded
from a version before this default stays in shadow mode, where API-key model
calls are refused, until an owner or administrator turns enforcement on under
**Settings → Protected credential enforcement**. This cannot be switched back.

The **Credentials** tab of **Settings** holds the organization's credential
settings: **Admitted provider hosts**, **Protected credential enforcement** and
**API-key credential approval**. Each has a help button that explains the rule
it enforces.

## Built-in key custody

The installer creates the signing, handover and retention keys on the server
itself, in root-owned configuration directories with restricted file modes. No
external Vault is required. Vault remains an optional integration that you
select explicitly. Include these keys in your backups; see
[backup and recovery](/docs/backup-and-recovery).

## Permission layers

- The GitHub App's repository selection and permissions control which source
  BranchRunner may read and where it may open draft pull requests.
- Each repository's draft pull request policy limits the allowed base branches.
- Organization roles control who may configure repositories, providers and
  workflows, and who may approve.
- Agent profiles request tool classes within installed platform limits.
- Network policy controls which services containers can reach.
- Human approvals control the plan, publication and other sensitive
  transitions. Publication approval needs a recent authenticator sign-in.
- A run step that uses an API key starts only after credential use is approved
  for that run in the run console. By default the person who asks can approve
  it with a recent authenticator sign-in; an owner can require a different
  person under **Settings → API-key credential approval**.

## Rotation

Rotate a provider key with **Replace key** on its connection, which saves a new
key revision. A Claude subscription token is valid for one year, and the
connection shows a renewal notice in its last 30 days; see
[Claude subscription sign-in](/docs/agent-providers#claude-subscription-sign-in).
Review which agent profiles use a connection before archiving it to avoid
unexplained run failures.

To withdraw a leaked or unwanted credential, use **Remove key** or
**Remove sign-in** under **Integrations**. It revokes the credential first and
then deletes it from custody. Also revoke the key at the provider. See
[agent providers](/docs/agent-providers).
